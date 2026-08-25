# MongoDB & Database Management - Notes

## Glossary of Key Terms

| Term | Definition |
|------|------------|
| **Express** | Node.js framework for building web applications and APIs |
| **MongoDB** | NoSQL database that stores data in flexible, JSON-like documents |
| **Mongoose** | ODM library for MongoDB and Node.js providing a schema-based solution |
| **Schema** | Defines structure and data types for documents in Mongoose |
| **Model** | A Mongoose structure used to interact with the database |
| **Collection** | A grouping of MongoDB documents, similar to a table in SQL databases |
| **Document** | A single record in a MongoDB collection |
| **Async/Await** | JavaScript syntax for handling asynchronous operations |
| **CRUD Operations** | Standard database operations: Create, Read, Update, Delete |
| **Atlas** | Cloud-based MongoDB service for hosting and managing databases |
| **Postman** | API client for testing and interacting with APIs |
| **JSON** | Data interchange format used to represent objects in web development |

---

## Lecture Overview
- Previous sessions covered Express.js: creating servers, CRUD operations, differences between PUT/PATCH/POST/DELETE
- **Today's agenda**: Exploring databases and how to integrate them with the server

---

## SQL Databases

### What is SQL?
- **SQL** = Structured Query Language
- Examples: Microsoft SQL Server, PostgreSQL, MySQL
- Data stored in **tables** (rows and columns)

### Schema
- **Fixed schema** - structure defined upfront
- Schema changes are **expensive**: heavy operation, takes time as table size grows

### When to use SQL?
- Applications with **fixed/consistent schema**
- Best for: Government apps, RTO licenses, bank accounts - data with consistent structure
- Operations remain same; only values change

---

## NoSQL Databases

### What is NoSQL?
- **Not only SQL** - still uses SQL-like querying but more flexible
- Supports **unstructured/heterogeneous data**
- Schema can change rapidly

### When to use NoSQL?
- Applications with **rapidly changing schema**
- Best for: Social media (Instagram), chat apps - random/variable data structure
- Handles heavy read/write/update operations efficiently
- SQL joins become performance bottlenecks in these cases

---

## Types of NoSQL Databases

| Type | Description | Example |
|------|-------------|---------|
| **Document-based** | Data stored as JSON-like documents | MongoDB, Firestore |
| **Key-value** | Simple key-value storage | Redis |
| **Graph** | Stores relationships between data | Neo4j |
| **Vector** | For similarity/search operations | (emerging) |

---

## MongoDB (Document Database)

### Structure
- Data stored as **JSON-like documents**
- Flexible schema - can store heterogeneous data easily
- Each document is a record with key-value pairs

### Real-world Example
- **Firebase Firestore**: Used in React course for storing authentication/details
- Also stores data as documents, not tables

### Why MongoDB?
- Hero of this course for MERN stack
- Handles unstructured data well
- No heavy table join operations
- Scales easily for web applications

---

## Redis (Key-Value Store)

### How it works
- Data stored as **key-value pairs** (like a dictionary/map)
- Simple and fast lookups

### Primary Use Case: Caching
- Saves frequently accessed data in memory
- Example: Mobile phone cache for app tabs
- Spotify uses **LRU (Least Recently Used)** cache technique

### Key Takeaways
- Extremely fast reads/writes
- Good for temporary data, sessions, caching
- If we have time: may implement Redis in project

---

## Graph Databases

### Neo4j
- Most popular graph database
- Stores **relationships between nodes** explicitly
- Best for: Social networks, recommendation engines, connected data

---

## SQL vs NoSQL - Decision Guide

| Factor | SQL | NoSQL |
|--------|-----|-------|
| Schema | Fixed | Flexible |
| Data Structure | Tables | Documents/Keys/Graphs |
| Schema Changes | Expensive | Easy |
| Joins | Supported | Not needed |
| Scalability | Vertical | Horizontal |
| Best For | Consistent data (govt, banking) | Rapidly changing data (social media, chat) |

---

## Key Concepts Summary
1. **SQL**: Tables with fixed schema - great for consistent structured data
2. **Adding columns in SQL**: Heavy operation - costs time as table grows
3. **Joins in SQL**: Can become performance bottleneck with large datasets
4. **NoSQL alternatives**: Document, Key-value, Graph, Vector databases
5. **MongoDB**: Document DB using JSON-like format
6. **Redis**: Key-value store primarily used for caching (LRU technique)
7. **Neo4j**: Graph database for relationship-heavy data

---

## MongoDB Basics (MongoDB NoSQL)

### Structure
- **Document**: JSON-like object stored in MongoDB
- **Collection**: Group of documents (analogous to a table in SQL)
- Built on document-based architecture, part of NoSQL family

### Database Naming
- Auto-named if unspecified: default = `'test'`
- Can explicitly name database during connection URL setup

---

## Mongoose Overview

### What is Mongoose?
- **ODM** (Object Data Modelling) library for MongoDB + Node.js
- Provides schema-based solution to model application data

### Schema vs Model
| Concept | Purpose |
|---------|---------|
| **Schema** | Defines document structure, default values, validators |
| **Model** | Compiled version of schema - interface for interacting with database |

### Naming Convention
- Models use **uppercase** naming as standard practice (e.g., `Course` not `course`)

---

## CRUD Operations with Mongoose

| Operation | HTTP Method | Mongoose Method | Description |
|-----------|-------------|-----------------|-------------|
| **Create** | POST | `CourseModel.create()` | Validates data against schema before inserting |
| **Read** | GET | `CourseModel.find()` | Retrieves all courses; uses async-await + error handling |
| **Update** | PUT | `findByIdAndUpdate()` | Finds by ID and updates the document |
| **Delete** | DELETE | `findByIdAndDelete()` | Finds by ID and removes the document |

### Key Practices
- All CRUD operations use **async-await** for asynchronous programming
- Proper error handling is essential for database operations
- Data is validated against schema rules before creation/update

---

## Connection Strings & Security

### What is a Connection String?
- Unique string that links your server to a MongoDB cluster
- Contains credentials: username, password, cluster address, authentication details

### Security Best Practices
- **Never expose** connection strings in client-side code
- Use environment variables (`.env`) for sensitive data
- Always secure the connection string - it provides full database access

---

## Express.js & Middleware Integration

### Server Setup
- Express is the foundational web server framework
- Handles incoming HTTP requests and outgoing responses

### Essential Middleware
- `express.json()` - parses incoming JSON request bodies
- Required for receiving POST/PUT data in correct format

---

## Practical Tasks / Homework
- Practice performing **PUT** and **DELETE** requests using Mongoose methods
- Implement full CRUD flow with proper async-await and error handling
- Store connection string securely in environment variables
