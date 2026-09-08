# MERN Stack - Social Media App: Session 2 Notes

## Session Overview
Continuation of building a social media app with focus on **password security**, **authentication**, and **session management**.

---

## 1. Problem: Plaintext Password Storage

### The Issue
- Previously, user passwords were stored in the database exactly as entered (plaintext)
- This is a critical security vulnerability — no one should access raw password data from the DB
- If the database is compromised, all passwords are exposed

**Solution:** Password **Hashing**

---

## 2. Password Hashing

### What is Hashing?
- A one-way mathematical function that transforms input (password) into a fixed-length string (hash)
- **Cannot be reversed** — you cannot get the original password back from a hash
- Industry-standard method for storing passwords

### Salt
- A random value added to the password before hashing
- Same password + different salt = completely different hash
- Prevents rainbow table attacks (pre-computed hash lookup attacks)
- Salt is stored alongside the hash in the database

### Rounds (Work Factor)
- bcrypt performs `2^n` transformations on the password, where `n` = rounds value
- Default: **10 rounds** → 2^10 = **1,024 transformations**
- With 12 rounds: 2^12 = **4,096 transformations**
- Higher rounds = more secure but slower; industry standard is moving toward 12
- Makes brute-force and cracking attacks computationally expensive

### bcrypt vs Argon2
| Feature | bcrypt | Argon2 |
|---------|--------|--------|
| Industry adoption | Widespread (most projects) | Emerging / newer |
| NPM package | `bcrypt` | `argon2` |
| Recommendation | Well-established | Better security profile |

### Hash Structure (bcrypt)
A bcrypt hash contains:
1. **Version** (`$2b$`) — bcrypt version identifier
2. **Rounds** — number of transformations used
3. **Salt** — stored within the hash
4. **Password hash** — the actual transformed password

---

## 3. Implementing Password Hashing in Registration

### Steps:
1. Import `bcrypt` (`bcrypt.hash()`, `bcrypt.genSalt()`)
2. Generate a salt with `bcrypt.genSalt()`
3. Hash the password with `bcrypt.hash(password, salt)`
4. Store **only the hash** in the database (NOT the plaintext)

```javascript
// In register controller
const salt = await bcrypt.genSalt(10);
const hashedPassword = await bcrypt.hash(req.body.password, salt);
user.password = hashedPassword;  // NOT req.body.password
await user.save();
```

---

## 4. Implementing Login with Password Comparison

### The Challenge
Since the password is stored as a hash, direct comparison won't work:
- `hash !== plaintext` always returns false

### Solution: bcrypt.compare()
bcrypt retrieves the salt and rounds from the stored hash, re-applies the same transformation to the user's input password, and compares the resulting hash.

```javascript
// In login controller
const user = await User.findOne({ email });  // Find user by email
if (!user) return res.status(404).json({ message: "User not found. Please register." });

const passwordMatch = await bcrypt.compare(req.body.password, user.password);
if (!passwordMatch) {
    return res.status(400).json({ message: "Wrong password." });
}

// Success — user is logged in
return res.status(200).json({ message: "Logged in successfully.", user });
```

### Flow:
1. User enters email + password → **validate** input
2. Check if **user exists** by email → 404 if not found
3. Use `bcrypt.compare()` to compare entered password with stored hash → Boolean
4. If match → return success (with user details or token)
5. If no match → 400 with error message

---

## 5. Cryptography Context

- **Cryptography** is the broader field covering encryption, hashing, and encoding
- Node.js has a built-in `crypto` module with algorithms like **SHA (SHA-256)**
- **SHA should NOT be used for passwords** — it's too fast (fewer transformations), making it easy to brute-force
- bcrypt/Argon2 are designed specifically for passwords with deliberate slowness

### Common Interview Questions:
- What is hashing vs encryption?
- Why store passwords as hashes, not plaintext?
- What is a salt?
- What is bcrypt? What are "rounds"?
- Best way to store passwords? → **bcrypt or Argon2 hashing**

---

## 6. Sessions & Authentication with JWT

### Session Persistence Problem
- After logging in, users shouldn't need to re-enter credentials every time they visit
- This requires **session management** — keeping the user "logged in" over time

### What is a Token?
- A piece of data that acts as temporary identity proof
- Analogy: Like a **boarding pass** at an airport — valid only for a specific time until the flight departs
- Similarly, tokens have an **expiration time** (e.g., 30 days)

### JWT (JSON Web Token)
- Format: `header.payload.signature` (three parts, URL-safe base64 encoded)
- Contains embedded data: user ID, expiration time (`exp`), initialization time (`iat`)
- Package: `jsonwebtoken` (`jwt.sign()`, `jwt.verify()`)

### Generating a JWT

```javascript
// utils/generateToken.js
const jwt = require('jsonwebtoken');

const generateToken = (userId) => {
    const secret = process.env.JWT_SECRET;  // Stored in .env file — NEVER expose
    const token = jwt.sign({ userId }, secret, { expiresIn: '30d' });
    return token;
};
```

**Key points:**
- Token is generated using the **user's ID** (MongoDB auto-generated `_id`)
- A **JWT_SECRET** acts as a signature/stamp to verify the token origin
- Always store the secret in a `.env` file, not in code
- Set expiration time (e.g., 30 days) — tracking starts from token creation time

### Decoding a JWT
Anyone can decode the payload (it's base64-encoded, **not encrypted**), but they cannot modify it without breaking the signature. Verify tokens on the server using `jwt.verify(token, secret)`.

---

## 7. Storing Tokens Securely: Cookies vs Local Storage

### ❌ Local Storage — NOT Recommended
- Any JavaScript running on the page can read the token
- Vulnerable to **XSS (Cross-Site Scripting)** attacks
- Tokens can be stolen by malicious scripts

### ✅ HTTP-only Cookies — Recommended
Cookies are a browser storage mechanism with configurable rules.

#### Setting an HTTP-only cookie:
```javascript
res.cookie('token', token, {
    httpOnly: true,   // JavaScript cannot access the cookie
    secure: true     // Cookie only sent over HTTPS
});
```

### Cookie Security Flags

| Flag | Purpose |
|------|---------|
| `httpOnly: true` | Prevents any client-side JavaScript from accessing the cookie — blocks XSS attacks |
| `secure: true` | Cookie is only sent over HTTPS connections — blocks **CSRF** (Cross-Site Request Forgery) attacks |

- **XS-Leaks**: Cookies/cookies leaking data across different sites
- **CSRF**: Attacker tricks a logged-in user's browser into making unauthorized requests

---

## 8. Authentication Flow Summary

```
User registers/logs in
    ↓
Generate JWT token (signed with JWT_SECRET)
    ↓
Store token in HTTP-only, secure cookie via res.cookie()
    ↓
Token sent automatically with every request by the browser
    ↓
Server verifies token using jwt.verify(token, secret)
    ↓
Extract user ID from verified token → find user in DB → return authenticated response
```

---

## 9. Todo for Next Session

- Ensure **register** and **login** routes generate a JWT token for both new and existing users
- Store the token in an HTTP-only cookie in both cases
- Test with Postman — verify cookies are set (check Cookies tab in Postman)
- Review XS-Leaks and CSRF attacks (demo in lab session)

---

## 10. Lab Assignments Notes

- Lab sessions now have **problem statements** (complete projects)
- Progressive build: Social Media App → E-commerce Platform
- Attendance depends on completing and getting lab assignments reviewed by TA
- Labs contribute **20%** of total course grade
- Aim to have at least 3 projects by the end of the course

---

## Key Takeaways

1. **Never store passwords in plaintext** — always hash with bcrypt
2. **bcrypt** uses salt + rounds; `bcrypt.compare()` for verification
3. **JWT** is used for session management — contains user ID + expiration
4. Store JWTs in **HTTP-only, secure cookies** (NOT localStorage)
5. Always validate tokens on the server side
6. SHA-256 and similar crypto algorithms are NOT suitable for password hashing
