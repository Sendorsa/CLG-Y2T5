# Deployment of a Fullstack App — Notes

## 1. Codebase Consolidation (Pre-Sockets)

### Problem Solved
- Every page was getting its own copy of ambient state → stale/inconsistent data
- **Solution:** Created one centralized Redux store with **Redux Toolkit slices** for each entity

### Key Changes
| Before | After |
|--------|-------|
| One big reducer | Separate slices (e.g., `postSlice`) |
| Manual store config | RTK (`configureStore`, `createSlice`) |
| Page-level state | Global store via actions & selectors |

> ✅ **Takeaway:** Use `createSlice` from `@reduxjs/toolkit` per domain. Each slice handles its own `state`, `reducers`, `extraReducers`.

---

## 2. Why Sockets? — The Limitation of HTTP

### HTTP Request-Response Cycle
```
Client → [Request] → Server → [Response] → Client   (connection dies)
```
- Connection **terminates** after every cycle
- To get new data → client must **poll** the server repeatedly

### Problem: Real-Time Notifications on REST
If User A likes/commments on User B's post:
- Request hits the server
- Server updates DB ✅
- But User B's client has **no way to know instantly** — HTTP is pull-only
- Workaround: Every few seconds → polling

---

## 3. Polling — Why It Doesn't Fit Well

### Short Polling
```
Client ──request──▶ Server ──data / empty──▶ Client (disconnect)
     ──wait 5s─────▶ Server ──empty──────────▶ Client (disconnect)
     ──wait 5s─────▶ Server ──data───────────▶ Client (disconnect)
     ──wait 5s─────▶ Server ──empty──────────▶ Client (disconnect)  ← wasted requests
```

| Aspect | Detail |
|--------|--------|
| Interval | Fixed (e.g., every 5 seconds) |
| Response if new data? | Returns data immediately |
| Response if no change? | Empty response — **wasted request** |
| Connection? | Terminates after each cycle |

**Verdict:** Wasteful. Most responses are empty. For real-time notifications — not optimized.

### Long Polling *(mentioned)*
- Server holds connection until data is available
- Still an HTTP workaround — sockets are better.

---

## 4. WebSockets — The Right Approach

### Core Concepts

| Term | Meaning |
|------|---------|
| **Full Duplex** | Both client & server can send & receive simultaneously over one connection |
| **Keepalive / Alive** | Connection stays alive continuously until someone closes it |
| **TCP** | Underlying transport protocol — data sent in sequence-numbered segments |

### How Socket.io Works
```
Client ◄══════=[Persistent Full-Duplex TCP Connection]══════▶ Server
         ▲              Can send/receive anytime              ▲
         └─────────── Always alive until closed ─────────────┘
```

### Real-Life Analogies
- **WhatsApp/Telegram instant messages** → always-on connection
- **Voice/Video calls** → bidirectional, continuous communication
- Disconnect only when one side ends it

---

## 5. Implementation — Socket.io Setup

### Step 1: Install Packages
```bash
# Server (backend)
npm install socket.io

# Client (frontend)  
npm install socket.io-client
```

### Step 2: Server-Side (Node.js + Express → `index.js`)

```js
const http = require('http');                              // Node built-in
const { createServer } = http;                           // Create HTTP server
const io = require('socket.io')(httpServer, {            // Attach Socket.io
  cors: {
    origin: 'http://localhost:5173',                     // Frontend URL
    methods: ['GET', 'POST'],
    credentials: true                                    // Pass cookies/tokens
  }
});

const httpServer = createServer(app);                    // Extract from Express `app`


```

> `server.listen(port)` instead of `app.listen(port)`

### Step 3: Listen for Socket Events

```js
io.on('connection', (socket) => {
  console.log('Client connected:', socket.id);            // Unique socket ID
  
  socket.on('disconnect', () => {
    console.log('Client disconnected:', socket.id);
  });
});
```

| Event | Trigger | Purpose |
|-------|---------|---------|
| `connection` | Client connects | Run on first connect (welcome, auth) |
| `disconnect` | Client disconnects | Cleanup, mark user offline |

### Step 4: Client-Side (`src/socket.js`)

```js
const socket = io('http://localhost:6000', {             // Server port
  credentials: true,\   / Pass cookies/tokens to server
  
});
// ⚠️ autoConnect: false  → we control when .connect() is called
export default socket;
```

---

## 6. Notification System — Using Sockets

### Feature Goal
- Real-time notifications (like ↔ comment ↔ follow)
- Red dot indicator in the navbar only when new notifications arrive
- Old notifications stay muted until user views them

### `Notification` MongoDB Model (`server/models/notification.model.js`)

```js
const notificationSchema = new mongoose.Schema({
  sender: { type: mongoose.Types.ObjectId, ref: 'User', required: true },
  receiver: { type: mongoose.Types.ObjectId, ref: 'User', required: true },
  type: {
    type: String,\   /  Enum: ['follow','like','comment'] |


  contentId: { type: mongoose.Types.ObjectId,\          // Referenced post/real/story

  isRead: { type: Boolean, default: false }             // New notifications stay `false`
});
```

| Field | Purpose |
|-------|---------|
| **`sender`** | Who triggered the action (ObjectId → User) |
| **`receiver** | Who gets notified **(ObjectId)** |
| **`type`** | Event type: `'follow'`, `'like'`, or `'comment'` (enum String) |
| **`contentId`** | ObjectId referencing the related Post/Real/Story |
| **`isRead`** | `false` = new/unread / `true` = read — determines red dot visibility |

### Why Store Notifications in DB? 🗄️
Even though sockets deliver instant events, we persist them because:
1. User may be **offline** when someone likes/comments/follows them
2. On next login → serve stored notifications from the database
3. Without persistence users would lose notification history

### Implementation Flow
```
User A (sender) ─▶ action (like/follow/comment) 
                       │
    ┌──────────────────┘
    ▼                           ▼
DB (NotificationModel)   ⚡ socket.emit('notification', data)
    │                           │
    └────── persisted ────▶ User B's client gets real-time event
                                    │
                            Red dot appears ✅
```

### Notification Logic Checklist
| # | Requirement | How to Handle |
|---|------------|----------------|
| 1 | Show red dot on navbar when new notifications arrive | Listen via socket → increment unread count |
| 2 | Hide red dot after viewing | Mark as read (DB) + clear badge (UI) |
| 3 | Only show *new* notifications in real-time | Check `isRead === false` before showing |
| 4 | Show total / new badges dynamically | Derived from unread count in Redux store |
| 5 | Handle offline users | Notifications stored in DB → loaded on login |

---

## 7. Next Steps (from the session)
- ✅ Create `Notification` model & store events
- 🔄 Next: Build notification controllers + emit events on like/follow/comment
- 🏁 Finalize real-time badge UI + Redux integration for notifications
