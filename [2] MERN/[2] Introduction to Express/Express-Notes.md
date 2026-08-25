# Express.js - Introduction Notes

## 1. What is Express.js?

**Express is a lightweight Node.js framework for building web applications and REST APIs.** Key features: suitable for creating web servers (and mobile app backends), lets you build your own API, very performant with async programming, and supports middleware.

---

## 2. Project Setup

### Initialize the project
```bash
npm init              # or npm init -y (skips prompts)
```

`npm init` creates a `package.json` file with:

| Field       | Description                                        |
| ----------- | -------------------------------------------------- |
| **name**    | Package/server name (defaults to directory name)   |
| **version** | Semantic versioning (major.minor.patch — e.g., `1.0.0`) |
| **description** | Text description of the server                 |
| **main**    | Entry point file (convention: `index.js`)          |
| **test**    | Test command (optional)                            |
| **repository** | Git repo URL                                     |
| **keywords**  | Searchable keywords                              |

**Versioning:** Servers use semantic versioning (e.g., `1.0.0` — major.minor.patch) to manage and communicate API changes.

### Install Express
```bash
npm install express
```
Express is listed as a **dependency** in `package.json`.

---

## 3. Creating a Basic Server

```javascript
const app = require('express')();   // Express function returns middleware — must be called with ()

app.listen(8000, () => {             // Start server on port 8000
  console.log('Server is running');
});
```

- `require('express')` returns a **function** — must be executed with `()` to get the app object.
- Convention: store it in a variable called `app`.
- Use **nodemon** for auto-restart on file changes:
  ```bash
  npm i -D nodemon
  npx nodemon index.js
  ```

---

## 4. Middleware

Middleware functions sit between the incoming request and the route handler. They can:

- Execute code
- Modify `req` / `res` objects
- End the request-response cycle
- Call the next middleware via `next()`

### Types of Middleware

| Type                | Binding                        | Note                                                         |
| ------------------- | ------------------------------ | ------------------------------------------------------------ |
| **Application-level** | `app.use(...)` on the Express instance | Most common — applies across the app or a specific mount path.              |
| **Router-level**     | `router.use(...)` on an `express.Router()` instance | Same API as application-level, but scoped to a router.       |
| **Error-handling**    | Four args: `(err, req, res, next)`              | Sourced by Express — built-in (`express.json()`, etc.) or installed via npm. |

### Built-in Middleware: `express.json()`

Express does **not** parse incoming JSON automatically (unlike browser `JSON.parse`). Use the built-in middleware to parse JSON request bodies:

```javascript
app.use(express.json());
```
This makes parsed data available at `req.body` for all subsequent routes.

---

## 5. Routes & HTTP Methods

### Defining a route

```javascript
app.METHOD(route, callback);        // e.g., app.get('/courses', …)
```

### HTTP methods (REST CRUD)

| Method   | Purpose                      | Example                 |
| -------- | ---------------------------- | ----------------------- |
| `GET`    | Retrieve data                | `GET /topics/courses`            |
| `POST`   | Create a new resource        | `POST /topics/courses`          |
| `PUT`    | Replace an entire resource  | `PUT /topics/courses/:id`       |
| `PATCH`  | Partially update a resource | `PATCH /topics/courses/:id`     |
| `DELETE` | Remove a resource            | `DELETE /topics/courses/:id`     |

---

## 6. RESTful API Fundamentals

**REST = Representational State Transfer** — an architectural style for building web services.

### Core principles

1. **Client–server architecture** — responsibilities are split; the server exposes endpoints, clients consume them.
2. **Statelessness** — every request must contain all information the server needs; no stored context between requests.
3. **JSON over HTTP** — communication uses the HTTP protocol with JSON as the standard data format.

### Route parameters

Dynamic URL segments let you identify **specific** resources:

```javascript
app.get('/courses/:id', (req, res) => {
  const course = courses.find(c => c.id === Number(req.params.id));
  res.json(course);
});
```

- Defined with a `:` prefix (`:id`).
- Accessible inside the handler via `req.params` (always **strings** — use `Number()` or `parseInt()` for numeric IDs).

### parseInt

`parseInt` parses a string and converts it into an integer in JavaScript. Useful when converting route parameters (which are strings) to numbers: `const id = parseInt(req.params.id);`

---

## 7. Full CRUD Example (In-Memory Store)

```javascript
let courses = [
  { id: 1, name: 'Java', instructor: 'John' },
  { id: 2, name: 'JavaScript', instructor: 'Jane' }
];

// GET all resources
app.get('/courses', (req, res) => {
  res.json(courses);
});

// GET a specific resource (route parameter)
app.get('/courses/:id', (req, res) => {
  const course = courses.find(c => c.id === Number(req.params.id));
  res.json(course);
});

// POST — create a new resource
app.post('/courses', (req, res) => {
  // Data arrives via req.body (parsed by express.json() middleware)
  courses.push(req.body);
  res.send('Course created');
});

// PUT — replace an entire resource
app.put('/courses/:id', (req, res) => {
  const c = courses.find(c => c.id === Number(req.params.id));
  Object.assign(c, req.body);       // overwrites all fields
  res.json(c);
});

// PATCH — partially update a resource
app.patch('/courses/:id', (req, res) => {
  const c = courses.find(c => c.id === Number(req.params.id));
  Object.assign(c, req.body);       // merges only provided fields
  res.json(c);
});

// DELETE — remove a resource
app.delete('/courses/:id', (req, res) => {
  const idx = courses.findIndex(c => c.id === Number(req.params.id));
  courses.splice(idx, 1);
  res.send('Course deleted');
});
```

---

## 8. Testing APIs with Postman

- Browsers only support **GET** requests natively.
- Use **Postman** (or similar tools) to test **POST**, **PUT**, **PATCH**, and **DELETE**.
- In Postman: pick the method → enter URL → go to **Body → raw → JSON** → write the payload → Send.

---

## 9. Key Takeaways

- Create `package.json` with `npm init -y`; list Express as a dependency (`npm i express`).
- A server is started via `app.listen(port, callback)`.
- Routes are defined with `app.METHOD(route, handler)` — the HTTP method determines the CRUD operation.
- **Route params** (`:id`) come through `req.params` (always strings); use `Number()` for numeric comparisons.
- **Request body** data comes through `req.body` — requires `express.json()` middleware.
- Middleware runs between the client request and the route handler; use `app.use()` to mount it.
- Use **Postman** to test non-GET HTTP methods.

---

## Homework

1. Research the difference between **PUT** (full replacement) and **PATCH** (partial update).
2. Write a **DELETE** route from scratch without looking at the notes.
