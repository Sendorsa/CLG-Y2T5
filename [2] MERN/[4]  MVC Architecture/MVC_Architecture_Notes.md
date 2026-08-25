# MVC Architecture - Structured Notes

## Overview
- MVC is a **design pattern** (not a programming concept)
- Represents an **architecture** that organizes the entire server
- Used to manage complex projects with independent components

## Real-Life Analogy: Restaurant

### Model (The Kitchen)
- Handles data and business logic
- Stores and retrieves information from the database
- Like the kitchen where the actual "cooking" happens

### View (The Menu/QR Code)
- The interface the user interacts with
- Displays information to the user
- Like a restaurant menu or QR code that shows dishes

### Controller (The Waiter)
- Mediates between Model and View
- Handles user requests and directs them appropriately
- Like a waiter taking orders and relaying them to the kitchen

## Practical Use Case: MERN Platform

### Application Scenario
Building a **paid course platform** with the following features:

1. **Course Management**
   - Display courses to learners
   - Allow users to browse available courses

2. **Purchase System**
   - Handle course purchases
   - Manage transaction data

3. **Independent Architecture**
   - Keep different functionalities separate
   - Make the system maintainable and scalable

### Why MVC?
- Manages everything independently
- Provides clean separation of concerns
- Makes the codebase easier to understand and maintain

## Key Takeaways

| Concept | Description |
|---------|-------------|
| Design Pattern | A reusable solution design approach, not a specific code |
| Architecture | Organizes the entire server structure |
| Separation of Concerns | Each component handles its own responsibility |
| Scalability | Easy to add new features without disrupting existing ones |

## MVC Flow

```
User Action -> Controller -> Model (Data/Logic) -> View (UI) -> User
```

1. User interacts with the **View**
2. **Controller** receives the request
3. Controller communicates with the **Model** for data/logic
4. Model processes and returns data
5. Controller sends data to **View**
6. View displays the result to the user

## Project Setup

### Server Initialization
- Use **Express** to create the server and define routes
- Connect to MongoDB using **Mongoose**
- Install required npm packages: `dotenv`, `express`, `mongoose`

### MVC Directory Structure
```
server/
├── models/       # Mongoose schemas & models
├── controllers/  # Business logic
├── routes/       # express.Router() URL mapping
├── middleware/   # Middleware functions
├── .env          # Environment variables
└── server.js     # Entry point
```

## .env File and Environment Variables

- **Purpose**: Store sensitive information separately from code (database URLs, JWT secrets, payment gateway keys)
- **Security Best Practice**: Never commit `.env` files to version control (add to `.gitignore`)
- **Implementation**:
  ```js
  require('dotenv').config();
  const DB_URL = process.env.DB_URL;
  ```
- **Common variables**: `DB_URL`, `JWT_SECRET`, `PAYMENT_GATEWAY_SECRET`

## Middleware

- **Purpose**: Functions that run between request and response
- Can update `req`/`res` objects, end requests early, or call `next()` to continue the chain
- **express.json()**: Parses incoming JSON payloads (required for handling POST/PUT bodies)
- Example:
  ```js
  const express = require('express');
  const app = express();
  app.use(express.json());
  ```

## Routes and Controllers

### Routes (Routers)
- Defined using `express.Router()`
- Map URLs to specific controller functions
- Example: `POST /api/products/create` → productController.create

### Controllers
- Handle incoming requests
- Contain **business logic**
- Process data using models
- Determine the response format

## CRUD Operations

| Operation | HTTP Method | Route Example          | Mongoose Method        |
|-----------|-------------|------------------------|------------------------|
| Create    | POST        | `/api/products/create` | `Model.create()`       |
| Read      | GET         | `/api/products`        | `Model.find()`         |
| Update    | PUT/PATCH   | `/api/products/:id`    | `findByIdAndUpdate()`  |
| Delete    | DELETE      | `/api/products/:id`    | `findByIdAndDelete()`  |

## Code Modularity

- **Importance**: Separating routes, models, and controllers into independent files
- **Benefits**:
  - Cleaner, more maintainable codebase
  - Easier scaling as the application grows
  - Better collaboration between developers
  - Simplified testing and debugging

## Glossary of Terms

| Term | Description |
|------|-------------|
| **env** | A file to store environment variables securely (`.env`) |
| **Express.js** | A Node.js framework for building web applications |
| **Mongoose** | A library for MongoDB object modeling in Node.js |
| **Middleware** | Functions that execute during the request-response cycle in Express.js |
| **MVC Pattern** | Model-View-Controller pattern for organizing code structure |
| **Route Parameters** | Parameters embedded in URL routes used for data operation identification (e.g., `:id`) |
| **CRUD Operations** | Create, Read, Update, Delete operations in web applications |
| **Controllers** | Functions to handle business logic in applications |
| **Schemas** | Structures defining the properties of data objects in Mongoose |
| **async/await** | Syntax for handling asynchronous operations in JavaScript |
| **Router** | An Express module to manage endpoints for applications |
| **process.env** | Object to access environment variables in Node.js |
