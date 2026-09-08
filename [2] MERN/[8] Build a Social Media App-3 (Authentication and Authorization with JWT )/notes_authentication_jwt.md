# MERN Project - Build a Social Media App (Part 3)
## Authentication & Authorization with JWT

### Session Overview

This session covered **authentication** implementation in a MERN stack social media application, specifically focusing on JWT (JSON Web Token) based cookie authentication. Everything was verified by creating a test user and restarting the server to confirm tokens are generated and stored correctly.

By end of this session: **backend auth is complete**. Next session shifts entirely to frontend.

---

### Previous Topics Covered

- **User Registration**: Registering a new user in the system
- **User Login/Logging**: Authenticating existing users
- **Password Hashing**: Securely hashing passwords before storage
- **Session-based Authentication**: Using tokens stored in cookies for auth state

---

### Key Concepts Discussed

#### 1. JWT Token Generation & Storage
- On successful registration/login, a token is generated
- Token is stored in **cookies** (not local storage) for better security
- Cookie has proper security flags (httpOnly, secure, sameSite)

#### 2. Security Best Practices
- Cookies with proper security attributes are not accessible via client-side scripts
- Token is only transferred over the network during authenticated requests
- Prevents XSS attacks compared to localStorage approach

#### 3. Authentication Flow
1. User registers/logs in
2. Server generates JWT token
3. Token stored securely in cookies
4. Client sends cookie with subsequent requests
5. Auth middleware validates the token
6. `next()` passes request to next controller/middleware
7. Valid user data is sent back in response

---

### Authentication Routes Implemented

| Route | Description |
|-------|-------------|
| `POST /register` | Register a new user |
| `POST /login` | Login with credentials |
| **Missing** | `POST /logout` — needs to be implemented |

---

### Homework / Action Items

#### 1. Implement Logout Route
- Create a `/logout` route
- Steps:
  1. Redirect the user (clear session/navigation)
  2. Remove the token from cookies

#### 2. Frontend Pages to Build (Before Next Session)
| Page | Purpose |
|------|---------|
| **Landing Page** | Entry/home page |
| **Sign Up Page** | User registration form |
| **Login Page** | User authentication form |
| **Home Feed/Feed Page** | Main content after login |

#### 3. Next Session Plan
- Start building the 4 frontend pages (React/Hono client)
- Integrate existing backend authentication routes
- Push code and notes to repository

---

### Authentication Flow Diagram

```
User Action          Server Action
─────────          ─────────────
Register/Login  →   Generate JWT Token
                    Store in Cookie (secure flags)
                    
Subsequent  →     Send cookie automatically
Request              Validate JWT
                    Attach user to request
                    Call next() → Controller
                    Respond with user data
```

---

### Important Notes

- Firebase/Clerk could simplify auth, but building from scratch teaches core concepts
- These authentication fundamentals apply to **100% of custom auth implementations**
- Three essential routes: Register, Login, Logout
- Cookie-based auth is more secure than localStorage due to httpOnly flags
