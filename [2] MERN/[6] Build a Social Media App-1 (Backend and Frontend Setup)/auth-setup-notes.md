# MERN Stack Social Media App - Session Notes

## Overview

Building a social media platform from scratch using the MERN stack (MongoDB, Express.js, React, Node.js). This session focuses on **authentication and authorization** setup.

---

## Project Features

- **Authentication**: Login / Sign-up pages with credentials
- **Profile Section**: Profile image, bio, user details
- **Content**: Post videos/images, upload stories
- **Dashboard**: Public feed showing posts & stories from followed users
- **Search**: Find other users
- **Chat**: Messaging between users
- **Follow System**: Followers / following

---

## Tech Stack & Key Concepts

- Authentication & Authorization built from scratch (no Firebase, no Clerk)
- JWT stored in browser for session persistence (users log in once)
- Password hashing (bcrypt mentioned for next session)
- Media files stored as URLs on **Cloudinary** (not directly in DB due to 512MB BSON size limit and best practices)

---

## Folder Structure (MVC Convention)

```
src/
├── models/
│   ├── user.model.js
│   ├── post.model.js
│   └── story.model.js
├── controllers/
│   └── user.controller.js
├── routes/
│   └── user.routes.js
└── index.js (main server file)
```

> **Naming convention matters**: model names must match exactly (case-sensitive) for proper cross-referencing between models.

---

## User Model Schema (Mongoose)

| Field | Type | Rules |
|---|---|---|
| `name` | String | Required |
| `email` | String | Required, Unique |
| `username` | String | Required, Unique |
| `password` | String | Required |
| `profileImage` | String | URL (Cloudinary), optional |
| `followers` | Array of ObjectId | References user model; stores IDs of users following this person |
| `following` | Array of ObjectId | Stores IDs of users this person follows |
| `posts` | Array of ObjectId | References post model |
| `stories` | Array of ObjectId | References story model |
| Timestamps | auto | `createdAt`, `updatedAt` (Mongoose default) |

**Why store IDs, not full objects, for followers/following?** — Better performance and avoids data duplication. User identity uses Mongoose `_id`.

---

## Registration Flow

### Route

```
POST /api/user/register
```

### Controller Logic (`user.controller.js`)

1. **Extract fields** from `req.body`: `name`, `username`, `email`, `password`
2. **Validate in try/catch block**

#### Validation Checklist

| Check | Condition | Status Code | Message |
|---|---|---|---|
| All fields present | Every required field has value | `400 Bad Request` | "All fields are required" |
| Password length | `password.length >= 6` | `400 Bad Request` | "Password must be >= 6 characters" |
| Username unique | `User.findOne({ username })` returns null | `409 Conflict` | "Username already exists" |
| Email unique | `User.findOne({ email })` returns null | `409 Conflict` | "Email already exists" |

#### Creating the User (after validations pass)

```js
const user = await User.create({ name, username, email, password });
return res.status(201).json({ message: "User registered", user });
```

### Error Handling

- Wrap DB operations in `try/catch`
- On crash → `res.status(500).json({ error: "Internal server error", details: err.message })`
- Server must **always stay running** even on errors

### Server Setup (`index.js`)

```js
app.use(express.json()); // parse incoming JSON bodies
app.use("/api/user", userRoutes); // mount routes after MongoDB connection
```

---

## HTTP Status Code Reference

| Code | Meaning | Use Case in Registration |
|---|---|---|
| `201` | Created | Successful registration |
| `400` | Bad Request | Missing fields, invalid data format |
| `409` | Conflict | Username or email already taken |
| `500` | Internal Server Error | Crash, DB unreachable |

---

## Client-Side vs Server-Side Validation

| Aspect | Client-Side | Server-Side |
|---|---|---|
| Checks | Format (email syntax, field presence) | Uniqueness in DB, data integrity |
| Bypassable? | Yes (DevTools) | No — always required |
| Recommendation | Both for best UX & security | Mandatory for real-world apps |

---

## Next Session Topics

- **Password hashing** using bcrypt (passwords must never be stored in plaintext)
- **JWT tokens** for login/ logout flows

---

*Key takeaway*: Always store passwords securely and media files on cloud storage — never in the database.
