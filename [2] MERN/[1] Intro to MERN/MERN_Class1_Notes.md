# MERN Stack - Class 1: Introduction to Node.js & Backend Development

## Course Overview

- **Total Classes**: 16 sessions
- **Tech Stack**: MongoDB, Express, Node.js (MERN) — React already covered previously
- **Goal**: Build full-stack applications with server, database, and client working together
- **Final Project**: Social media application
- **Architecture Pattern**: MVC (Model-View-Controller)
- **First 4-5 classes**: Fundamentals of Express, Node.js, MongoDB setup
- **Remaining classes**: Hands-on project building

---

## Part 1: Node.js — Introduction & Environment

### What is Node.js?
- JS runtime environment outside the browser (on local OS)
- Provides built-in modules the browser `window` object does not expose
- Powered by the **V8 engine** — the same JavaScript engine that runs in Google Chrome
- Combines V8 engine with C++ code to create a standalone runtime for local machines
- **Cross-platform**: Like Java or Python, Node.js allows JavaScript to run across different platforms (Windows, macOS, Linux)

### Setting Up a Node Project
```bash
npm init          # Initialize package.json interactively
npm init -y       # Initialize with defaults
```
- **`package.json`**: Configuration/metadata file for the project
- **Entry point**: Conventionally `index.js` (main file where server starts)

---

## Part 2: File System (`fs`) Module

The **`fs` module** is Node.js's built-in module for file system operations such as read, write, append, and delete.

### Key Methods

| Term | Description | Method |
|---|---|---|
| Read File Sync | Synchronous reading of files, returning data in hexadecimal format by default | `fs.readFileSync(file)` |
| Write File Sync | Method to write data synchronously to a file, overwriting existing content | `fs.writeFileSync(file, data)` |
| Append File Sync | Method to add data to the end of a file without removing existing content | `fs.appendFileSync(file, data)` |
| Unlink Sync | Synchronous method to delete a file in Node.js | `fs.unlinkSync(file)` |

### ⚠️ Problem with Synchronous Operations
- JavaScript has a **single-threaded** event loop
- Heavy operations (e.g., reading large files with 7+ lakh lines) **block** the main thread
- All subsequent code waits — called **blocking behavior**
- Example: 7-8 seconds processing time makes apps unresponsive

### Asynchronous File Operations (Non-blocking)
```js
const fs = require('fs');

fs.readFile('f1.txt', (err, data) => {
    if (err) throw err;
    console.log(data);
});
```

- Heavy tasks are offloaded to a wait space → continues executing main thread
- Once complete, result is picked from the **task queue** (callback queue)
- **Event loop** checks if call stack is empty, then pushes callback into it for execution

---

## Part 3: Callbacks & Event Loop Architecture

### Key Terms

| Term | Description |
|---|---|
| **Asynchronous Operation** | Execution approach allowing non-blocking code execution using callbacks or promises. Heavy tasks are offloaded to a wait space, allowing the main thread to continue executing. |
| **Callback** | A function passed as an argument to another function, to be invoked later once an operation completes. Error-first pattern: `(err, data) => {}`. |
| **Blocking** | A synchronous operation that halts further execution until it completes. All subsequent code waits during blocking behavior. |

### How Async Works Behind the Scenes
1. **Call Stack**: Executes synchronous code one-by-one
2. **Node APIs**: Offload async tasks (e.g., file I/O)
3. **Task Queue / Callback Queue**: Holds completed async task callbacks
4. **Event Loop**: Monitors call stack; when empty, pushes from task queue to call stack

### Error-First Callbacks
- Standard callback pattern: `(err, data) => {}`
- **Error comes first**, data second
- Always check for errors before handling data
```js
fs.readFile('file.txt', (err, data) => {
    if (err) {
        console.error('File does not exist or cannot be read');
        return;
    }
    console.log(data); // handle data only if no error
});
```

### Key Concepts to Revise
- Callbacks, Promises, Async/Await — essential for communicating between external systems (client → server → database)

---

## Part 4: HTTP Protocol & REST Architecture

### What is REST?
- **REST** = Representational State Transfer
- Defines communication between independent systems (client ↔ server ↔ database)
- Uses **JSON** as universal data format (alternatively XML possible)
- Communicates via **HTTP protocol** — defines request/response lifecycle

### HTTP Methods (CRUD Operations)
| Method | Operation |
|---|---|
| `GET` | Read data from server |
| `POST` | Create new resource |
| `PUT` | Update a resource |
| `DELETE` | Delete a resource |
| `PATCH` | Partial update (discussed later) |

### HTTP Status Codes
| Code | Meaning |
|---|---|
| `200` | OK — success |
| `201` | Created |
| `204` | No Content |
| `1xx`, `3xx`, `4xx`, `5xx` | Various other statuses (info, redirect, client error, server error) |

---

## Part 5: Building a Server with the HTTP Module

The **`http` module** is Node.js's built-in module for building and managing HTTP servers and handling requests/responses.

### Creating a Basic Server
```js
const http = require('http');

const server = http.createServer((req, res) => {
    res.end('Welcome to my server!');
});

server.listen(8000, () => {
    console.log('Server started at port 8000');
});
```

### Accessing the Server
- Navigate to `http://localhost:8000` in browser
- Server responds with the text passed to `res.end()`

### Multiple Routes Using Switch Statement
```js
const http = require('http');

const server = http.createServer((req, res) => {
    console.log(req.url);  // logs requested URL path
    
    switch (req.url) {
        case '/':
            res.end('Home page. Welcome to my cell.');
            break;
        case '/about':
            res.end('This is my about page.');
            break;
        case '/contact':
            res.end(`Contact page HTML`);
            break;
        default:
            res.end('404 - Not Found');
            break;
    }
});

server.listen(8000, () => {
    console.log('Server started at port 8000');
});
```

### Request Object
- `req.url` — the path of the incoming request
- Default route is `/`; browser also requests `/favicon.ico` implicitly
- Every endpoint corresponds to a route in `req.url`

### Server-Side Rendering
- Sending an entire HTML page as a response via template literals
- Concept: UI is rendered and served from the **server** (not the client)
- Called **server-side rendering** (SSR)

### Drawbacks of Bare HTTP Module
- Manual route handling with if/else or switch
- Must add `break` after every `res.end()` to prevent crashes
- Edge cases must be handled manually
- Becomes complex quickly for larger applications

---

## Part 6: Introduction to Express Framework

### Why Express?
- Solves the complexity of the raw HTTP module
- Simplifies routing, middleware, and response handling
- "Express is a framework for server-side development, similar to how React is a framework for UI"
- Used from the **next class** onward for server creation instead of bare `http`

---

## Key Takeaways

1. **Node.js** enables JavaScript on the backend via its runtime and modules (`fs`, `http`)
2. **Synchronous operations block** the single thread → use asynchronous patterns for performance
3. **Callbacks** are error-first: `(err, data)` — foundational to understanding Promises/Async-Await
4. **REST** uses HTTP methods + JSON for client-server communication
5. **HTTP module** is powerful but verbose; **Express framework** abstracts complexity for faster development
6. Next class: Build servers with Express instead of raw `http`

---

## Homework / Preparation for Next Class

- Revise: **Asynchronous programming**, **Callbacks**, **Promises**
- Practice: Create a basic Node project, experiment with `fs` and `http` modules
- Resource: Study HTTP status codes reference documentation
