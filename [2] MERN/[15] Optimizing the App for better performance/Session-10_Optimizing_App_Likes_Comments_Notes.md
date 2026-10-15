# Session 10: Adding Likes, Comments & Feeding Content to Client

> **Context:** Continuation of building a Social Media App (MERN Stack). Previous sessions covered Post/Reel creation. This session covers: retrieving posts/reels with author data, wiring client-to-backend, likes (toggle), and comments (separate model + gateFilter pattern).

---

## 1. Getting Posts & Reels — Populating Author Data

### Backend (`post.controller.js`)

```js
exports.getAllPosts = async (req, res) => {
    try {
        const posts = await Post.find().populate('author');
        if (!posts || posts.length === 0) {
            return res.status(404).json({ message: 'No posts found' });
        }
        res.status(200).json(posts);
    } catch (error) {
        res.status(500).json({ message: 'Server error', error });
    }
};
```

* `Post.find()` — fetches all posts.
* `.populate('author')` — replaces the `author` ObjectId with full user document (name, profileImage, etc.).

> **Same pattern applies to Reels:** `Reel.find({}).populate('author')` in `reel.controller.js`.

### Backend route (`post.routes.js`)

```js
router.get('/get-all-posts', authMiddleware, getAllPosts);
```

### Client (Axios)

```js
// Fetch posts + reels → merge → sort by date
const [allPosts, allReels] = await Promise.all([
    axios.get('http://localhost:8089/api/posts/get-all-posts'),
    axios.get('http://localhost:8089/api/reels/get-all-reels')
]);

// Merge into one feed array + sort descending (latest first)
const items = [...allPosts.data, ...allReels.data];
items.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
setFeed(items);
```

---

## 2. Creating Posts/Reels from Client — Form Data + Axios

When a user selects an image (post) or video (reel) and adds a caption:

1. **Validation** — button enabled only when `caption` and `selectedFile` are both non-empty.
2. **Build FormData:**

```js
const formData = new FormData();
formData.append('mediaType', contentType);       // 'post' or 'reel'
formData.append('file', selectedFile);
formData.append('caption', captionText);
```

3. **Dispatch to correct endpoint** based on `contentType`:

| contentType | Endpoint                 |
| ----------- | ------------------------ |
| `'post'`    | `POST /api/posts/create-post` |
| `'reel'`    | `POST /api/reels/create-reel` |

```js
const endpoint = contentType === 'post' 
    ? '/api/posts/create-post' 
    : '/api/reels/create-reel';

await axios.post(`http://localhost:8089${endpoint}`, formData, {
    headers: { 'Content-Type': 'multipart/form-data' }
});
```

4. **Refresh feed** after creation — re-call `get-all-posts` + `get-all-reels`, merge, and sort by `createdAt` descending.

---

## 3. Like Feature (Toggle Like / Unlike)

### Model Change — Posts model

Add a `likes` array field to the Post schema:

```js
likes: [{ type: mongoose.Types.ObjectId, ref: 'User' }]
```

* Array length → number of likes.
* Each element → ObjectId of user who liked.

### Controller (`post.controller.js`) — `updateLike`

```js
exports.updateLike = async (req, res) => {
    const { id } = req.params;          // post ID
    const userId = req.user._id;         // from auth middleware

    const post = await Post.findById(id);
    if (!post) return res.status(404).json({ message: 'Post not found' });

    // Check if already liked: compare as strings to avoid type mismatch
    const alreadyLiked = post.likes.some(like => like.toString() === userId.toString());

    if (alreadyLiked) {
        // UNLIKE — remove user ID from array
        post.likes = post.likes.filter(like => like.toString() !== userId.toString());
    } else {
        // LIKE — add user ID to array
        post.likes.push(userId);
    }

    await post.save();   // ← easily forgotten!

    return res.status(200).json({
        message: alreadyLiked ? 'unliked' : 'liked',
        likeCount: post.likes.length
    });
};
```

### Route (`post.routes.js`)

```js
router.post('/:id/like', authMiddleware, updateLike);
```

### Client-side Like Button

* On heart/thumbs-up click → `POST http://localhost:8089/api/posts/${itemId}/like`.
* Use `item._id` from the feed state.
* Toggle UI (filled vs outline heart) and update like count locally or via response.

> **Bug encountered during class:** Forgot `await post.save()` — unlike never persisted because Mongoose doesn't auto-save modified arrays.

### Homework — Show who liked

Use user IDs stored in `likes` array to fetch names/profile images (populate likes → `{ path: 'likes', select: 'name profileImage' }`).

---

## 4. Comments Feature — Separate Model + Gate Filter Pattern

### Why a separate Comment model?
Comments have their own properties (author, text, timestamp) that don't belong inline inside Posts/Reels per normalization best-practice.

### Comment Schema (`comment.model.js`)

```js
const commentSchema = new mongoose.Schema({
    author: { type: mongoose.Types.ObjectId, ref: 'User' },   // who commented
    text:   { type: String, required: true },                 // comment content
    post:   { type: mongoose.Types.ObjectId, ref: 'Post' },   // null for reel-only comments
    reel:   { type: mongoose.Types.ObjectId, ref: 'Reel' },   // null for post-only comments
    createdAt: { type: Date, default: Date.now }              // when commented
}, { timestamps: true });

module.exports = mongoose.model('Comment', commentSchema);
```

### GateFilter Pattern (`gateFilter.js`)

Encapsulates "which collection should this comment target?" logic in one reusable function.

```js
const gateFilter = (type, id) => {
    const filter = {};
    if (type === 'post') filter.post = id;
    else if (type === 'reel') filter.reel = id;
    else return null;   // invalid type
    return filter;
};

module.exports = gateFilter;
```

### Controller (`comment.controller.js`) — `createComment`

```js
const Comment = require('../models/comment.model');
const gateFilter = require('../gateFilter');

exports.createComment = async (req, res) => {
    const { type, id } = req.params;
    const { text } = req.body;
    const author = req.user._id;

    const filter = gateFilter(type, id);
    if (!filter) return res.status(400).json({ message: 'Invalid content type' });

    // Validation
    if (!text || text.length > 500) {
        return res.status(400).json({ message: 'Text must be between 1-500 characters' });
    }

    const newComment = await Comment.create({
        author,
        text,
        ...filter   // spreads { post: id } or { reel: id }
    });

    return res.status(201).json({ message: 'Comment created', comment: newComment });
};
```

### Routes (`comment.routes.js`)

```js
const router = require('express').Router();
const { createComment, getComments } = require('./comment.controller');
const authMiddleware = require('../middleware/authMiddleware');

router.post('/:type/:id/', authMiddleware, createComment);

// GET by type + id → fetches comments for that post or reel
router.get('/:type/:id', getComments);

module.exports = router;
```

Register in `index.js`: `app.use('/api/comments', commentRoutes);`

### Client-side Comment Button

* On click → open a text input.
* On submit → `POST http://localhost:8089/api/comments/${type}/${itemId}/comment` with `{ text }`.
* Merge fetched comments into feed items (or render inline below the post).

---

## 5. Full Data Flow Summary

```
[Client] Post/Reel Button click
    │
    ├── Has caption? ─No──► button disabled
    └── Has file?    ─No──► button disabled
    │
    ▼ Yes / Yes (both valid)
[Form Data: mediaType, file, caption]
    │
    ├── content-type = 'post'  → POST /api/posts/create-post
    └── content-type = 'reel'  → POST /api/reels/create-reel
    │
    ▼ Backend creates document with { author (ref), caption, image/video, contentType='post'|'reel' }
[Axios response returns created object with _id]
    │
    ▼ Refresh feed: GET /posts/get-all-posts + GET /reels/get-all-reels
            merge → sort by createdAt desc → setFeed()
    │
[User sees post/reel in feed with author name + profileImage (populated)]
    │
    ├── ❤ Like button click  → POST /api/posts/:id/like   (toggle)
    └── 💬 Comment button click → POST /api/comments/:type/:id/comment (text)
```

---

## Key Takeaways & Patterns Used

| Concept | How it was applied |
|---------|-------------------|
| **`.populate()`** | Posts/Reels return full author doc instead of bare ObjectId |
| **Form Data + Axios** | Multipart form submission for media upload from React client |
| **Dynamic endpoints** | `contentType` routes data to correct controller (`/post/create-post` vs `/reel/create-reel`) |
| **Feed merge + sort** | Fetch both collections → combine array → `sort((a,b) → b.createdAt - a.createdAt)` |
| **Toggle (like/unlike)** | `some()` check → `push()` or `.filter()` on likes array → `await post.save()` |
| **Separate model** | Comments extracted to own `Comment` schema for normalization |
| **GateFilter pattern** | Single function decides `{ post }` vs `{ reel }` filter — keeps controller logic clean |
| **Validation** | Auth middleware, text length limits (1–500 chars), empty-field checks |

## Bugs Encountered in Class & Fixes

* **Unlike not working:** Forgot `await post.save()` — Mongoose arrays don't auto-save. Fixed by adding `save()`.
* **Filter returning `undefined`:** Missing `return` inside `gateFilter`. Fixed by explicitly returning the built filter object.
* **Comment creation failing (500):** Routes registered in `comment.routes.js` but missed registration in `index.js`. Added `app.use('/api/comments', commentRoutes)`.

## Homework / To-Do

1. Wire like/unlike buttons in React — connect to `POST /api/posts/:id/like`, update UI state.
2. Wire comment input/buttons — connect to `POST /api/comments/:type/:id/comment`, render comments inline.
3. Populate who liked: `Post.find().populate({ path: 'likes', select: 'name profileImage' })` → display usernames under posts.
4. **Next session:** Stories feature (create + expire resources), notification settings, wrapping up the feed.
