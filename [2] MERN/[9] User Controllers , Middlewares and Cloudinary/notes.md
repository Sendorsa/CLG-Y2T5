# Comprehensive Revision Notes: Building and Securing a Full Stack Application

> Logo Scaler Companion · Beta

---

## Overview

In this class, we covered the integration of backend authentication mechanisms with a frontend application using React. We explored user authentication routes, implemented client-server communication, and handled user authentication states. Additionally, we discussed handling Cross-Origin Resource Sharing (CORS) issues, leveraging React Router for managing different pages, and setting up asynchronous API calls using Axios.

---

## 1. Backend Authentication Recap

### User Routes

Three primary user routes were discussed:

| Route | Purpose |
|-------|---------|
| `/register` | Create a new user account |
| `/login` | Authenticate existing user and return token |
| `/me` | Get current authenticated user's data (protected) |

### Middleware

- **AuthMiddleware** gets the token from cookies, verifies it, and sets the user in the request.
- Registered users can be verified via tokens, allowing seamless access to routes without repeated logins.

---

## 2. Setting Up the Frontend with React

### Initializing a React App with Vite

```bash
# Using Vite (recommended for modern React apps)
npm create vite@latest my-app --template
```

**Tools**: Vite was used to initiate a new React app with plain JavaScript.

**Project Structure**: Components were organized under the `src` directory, with a focus on keeping the structure clean by removing default boilerplate code and styles.

### Creating Frontend Pages

**Core Pages**: Created Signup, Login, Home pages, along with a Landing page as components under the `pages` directory.

**Project Structure Example:**
```
src/
├── pages/
│   ├── Landing.jsx
│   ├── Signup.jsx
│   ├── Login.jsx
│   └── Home.jsx
├── components/
├── axiosInstance.js       # Centralized Axios instance
└── App.jsx                # Main routing setup
```

### Routing Setup

Utilized React Router for setting up navigable routes for each page, implementing Route components for navigation.

```js
// In App.jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Landing from "./pages/Landing";
import Signup from "./pages/Signup";
import Login from "./pages/Login";
import Home from "./pages/Home";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Landing />} />
        <Route path="/signup" element={<Signup />} />
        <Route path="/login" element={<Login />} />
        <Route path="/home" element={<Home />} />
      </Routes>
    </BrowserRouter>
  );
}
```

---

## 3. Implementing Authentication on the Frontend

### Signup Page

**State Management**: Leveraged React's `useState` to manage form inputs for name, username, email, and password.

**Form Handling — Multi-field Inputs**:

```js
const [form, setForm] = useState({ name: "", username: "", email: "", password: "" });

const change = (e) => {
  setForm({ ...form, [e.target.name]: e.target.value });
};
```

**How it works**:
- `form` contains all previously entered values
- `[e.target.name]` uses the input field's `name` attribute as the dynamic key (must match object keys: `name`, `username`, `email`, `password`)
- `e.target.value` provides the new value for that key
- Previous data stays intact via spread operator

**onSubmit Handler**:

```js
const submit = async (e) => {
  e.preventDefault();
  try {
    const response = await axiosInstance.post("/users/register", form);
    console.log(response.data);
    toast.success("Registered");
  } catch (error) {
    console.error(error);
    toast.error(error.response?.data?.message || "Something went wrong");
  }
};
```

- `form` as second parameter = **payload** (data sent with the request)
- Attach to all inputs: `onChange={change}`
- Attach to form: `onSubmit={submit}`

### Login Page

**Authentication Flow**: Similar setup as the Signup page but focuses on collecting email and password.

**Navigation**: On successful login, users navigate to the Home page via React Router's `useNavigate` hook.

```js
import { useNavigate } from "react-router-dom";

const navigate = useNavigate();

const submit = async (e) => {
  e.preventDefault();
  try {
    const response = await axiosInstance.post("/users/login", form);
    toast.success("Logged in");
    navigate("/home");
  } catch (error) {
    toast.error(error.response?.data?.message || "Login failed");
  }
};
```

---

## 4. Ensuring Server and Client Communication

### Axios Instance Setup — `src/axiosInstance.js`

**Why**: Every server call needs the same base configuration. Creating an instance avoids repeating these settings.

```js
import axios from "axios";

const axiosInstance = axios.create({
  baseURL: "http://localhost:8089",       // Server address & port
});

axiosInstance.defaults.withCredentials = true;   // Send cookies/session data
axiosInstance.defaults.headers.post["Content-Type"] = "application/json";

export default axiosInstance;
```

### Three key configurations:

| Config | Purpose | Value/Setting |
|--------|---------|---------------|
| `baseURL` | Server address for all API calls | `http://localhost:8089` (or your port) |
| `Content-Type` header | Data format for communication | `application/json` |
| `withCredentials` | Send cookies/tokens with each request | `true` |

### Usage in components:
```js
import axiosInstance from "axiosInstance";

// Then: await axiosInstance.post("/users/register", form)
```

### Server Port Reference

Instructor's server runs on port **8089** (`localhost:8089`), not the more common `8080`. Verify your actual server port before setting `baseURL`.

---

## 5. Handling CORS Issues

### Error Encountered

> "Access to XMLHttpRequest... blocked by Cross-Origin Policy. Response to pre-flight request doesn't pass access control check."

### Why it happens:
- **Same Origin Policy**: Browsers block requests from different domains/ports by default
- Client runs on `http://localhost:5173`, server on `http://localhost:8089` → different origins
- Before making the actual POST request, browser sends a **preflight (OPTIONS) request** to check permissions
- CORS blocks that preflight if not configured

### Solution: Install and configure `cors` on the **server side**

```bash
# On server
npm install cors
```

```js
// server app.js / index.js
const cors = require("cors");
app.use(cors({ origin: "http://localhost:5173" }));
// or with credentials:
// app.use(cors({ origin: "http://localhost:5173", credentials: true }));
```

### Common pitfalls:
- **Missing trailing slash** on origin (`/` vs no `/`) — can crash requests
- **Port mismatch** — verify client and server are running on expected ports
- **Wrong Content-Type header spelling** — `application/json` (not `aplication`)

---

## 6. Sign-Up Flow Summary

```
User fills form (name, username, email, password)
        │
        ▼
onChange updates state via spread operator: ...form, [name]: value
        │
        ▼
onSubmit → POST /users/register with form data as payload
        │
        ▼
Axios instance sends request ──→ server (handles CORS, middleware, validation)
        │
        ▼
Success → "User registered successfully" → log in page
Error   → error message via toast
```

---

## 7. Styling with Tailwind CSS

**Integrating Tailwind**: Tailwind CSS was configured in the React app for rapid styling without writing custom CSS, following setup instructions from Tailwind documentation.

### Quick Setup:
```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```js
// tailwind.config.js
module.exports = {
  content: ["./src/**/*.{js,jsx}"],
  theme: { extend: {} },
  plugins: [],
};
```

```css
/* index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

## 8. Developing Better User Experience

### Error Messaging — UI Feedback

**Task**: Display server-side validation errors on the UI for better UX, rather than relying on console outputs.

- Use `toast.error()` from toast libraries (e.g., react-hot-toast) to show error messages returned by the server
- Access via: `error.response?.data?.message` or similar paths depending on server response format

### Loading Indicators — UI Enhancements ✓ Homework Task

**As a homework task**: Implement loading indicators during asynchronous operations to improve perceived performance.

**Approach**:
```js
const [loading, setLoading] = useState(false);

const submit = async (e) => {
  e.preventDefault();
  setLoading(true);  // Show spinner/loading state
  try {
    await axiosInstance.post("/users/register", form);
    toast.success("Registered");
  } catch (error) {
    toast.error(error.response?.data?.message || "Something went wrong");
  } finally {
    setLoading(false);  // Hide spinner
  }
};
```

**UI Enhancement Ideas**:
- Disable submit button while loading: `<button disabled={loading}>`
- Show spinner icon during async operations
- Display "Loading..." text or skeleton screens

---

## Conclusion

This class provided a comprehensive walkthrough of integrating authentication mechanisms between backend and React frontend applications. Key takeaways included:

- Setting up API routes (`/register`, `/login`, `/me`)
- Handling front-end state for authentication
- Solving CORS issues for seamless client-server interaction
- Implementing Axios instances for centralized HTTP configuration
- Using React Router for page navigation
- Enhancing UI/UX with error handling and loading indicators

This setup is fundamental for building secure and interactive full-stack applications that manage user data effectively.

---

## Glossary

| Term | Definition |
|------|-----------|
| Token | A unique identifier used to authenticate a user's session |
| Protected Page | A page that requires user authentication to access |
| Public Route | Routes accessible without authentication |
| Protected Route | Routes that require authentication to access |
| Context | Shared data structure in React for global information |
| Auth Provider | Component to wrap pages and provide authentication state |
| User Data | Information about the currently authenticated user |
| useEffect | React hook that runs on render or updates |
| useParams | React Router hook to access URL parameters |
