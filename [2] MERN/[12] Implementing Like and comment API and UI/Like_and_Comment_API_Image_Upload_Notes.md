# Implementing Like and Comment API & UI — Image Upload Notes

---

## Terms & Concepts Quick Reference

| Term | Meaning |
|---|---|
| Profile Page Navigation | Users can navigate to their own profile and other users' profiles |
| Follow Feature | Allows users to follow and unfollow others using API routes |
| Follow User Controller | Handles API requests to follow users |
| Unfollow User Controller | Handles API requests to unfollow users |
| API Endpoint | A designated URL for accessing server resources (e.g., `/api/follow`, `/api/unfollow`) |
| useParams Hook | React Router DOM hook used to extract parameters from the URL |
| useState (React Hook) | Manages state within functional React components |
| Multer Middleware | Handles multipart form data, particularly for file uploads |
| RAM Storage in Multer | Stores files temporarily in RAM as a buffer for fast access |
| Preview Image State | Stores the URL of the image selected for preview purposes |
| Buffer in Multer | Represents the raw image data processed and ready for storage or transfer |

---

## Topic Overview
- Implementing **Follow Feature** (follow/unfollow toggle, profile navigation, state management)
- Client-side image preview for profile photo upload
- Why raw files should not be stored in the database
- Setting up **Multer** middleware for multipart form data handling
- Integrating **Cloudinary** for cloud-based media storage
- Storing Cloudinary URLs in the database as permanent references
- Implementing Like API (create, delete, toggle) with React Query
- Implementing Comment API (create, read, delete) with nested replies
- Frontend UI components for likes and comments display

---

## 0. Implementing Follow & Unfollow Functionality

### Profile Navigation

- Users can access their own profile or navigate to other users' profiles by clicking on username/profile picture links.
- The route uses URL parameters via React Router DOM's `useParams` hook to identify the target user from the URL.

### Backend Setup

- Created dedicated **controllers** for follow and unfollow actions in the server-side codebase.
- Controllers accept the **target user's ID** sent from the client-side API request.
- The controller checks if the current logged-in user is already following the target profile:
  - If **not followed** → adds a follow entry (persisted to database).
  - If **already following** → removes the follow relation (unfollow operation).

### Client-Side Workflow (Step-by-step)

1. User navigates to another user's profile page using link components or profile picture/username routing.
2. Once loaded, the app checks whether the current user already follows that profile.
3. A Follow / Unfollow toggle button renders based on the check:
   - If `isFollowing === true` → displays "Unfollow"
   - If `isFollowing === false` → displays "Follow"

4. When the user clicks either button, an Axios call is made to the corresponding backend endpoint (with the profile's ID).
5. After the Axios promise resolves:
   - The local state is updated for instant UI change (optimistic update).
   - Database record changes are applied and confirmed by the server response.
6. Follower/following counts displayed on the profile also increment or decrement dynamically.

### Follow/Unfollow Workflow Summary

| Step | Action | What Happens |
|---|---|---|
| 1 | Navigate to target profile | Loads the user info + current follow status via `useParams` |
| 2 | Render Follow/Unfollow button | Button text depends on value returned from state/API |
| 3 | User clicks button | Sends Axios `POST /api/follow` or `POST /api/unfollow` request along with the target user's ID |
| 4 | Controller receives request | Checks existing relation → adds follow entry OR removes it from database |
| 5 | Server responds (success) | Axios call resolves, UI updates to show toggle state correctly |
| 6 | Display updates | Follower count increases / decreases; button toggles accordingly |

### Key Concepts Covered

- **`useParams` hook** — Used to extract profile ID from the URL parameter and target other users' profiles.
- **Axios requests (POST)** — Sends follow/unfollow action along with the profile user ID to the backend.
- **Boolean logic for UI toggle** — `isFollowing ? "Unfollow" : "Follow"` renders the correct button text.
- **Database manipulation** — The server-side controller adds a follower entry when someone follows, and removes it on unfollow.
- **Live state & count updates** — Both the UI button and live follower/following counts update immediately after API success.

---

## 1. Client-Side Image Preview (Frontend)

### File Input & State Management

- Used `<input type="file">` to allow users to select images from their local system.
- On file selection, the file is converted to a URL via `URL.createObjectURL(file)` → generates a **temporary client-side URL**.
- The generated URL is stored in component state using React's `useState` hook for preview purposes (e.g., `const [previewImage, setPreviewImage] = useState(null)`).

### Flow

1. User clicks "Edit Profile" → upload form opens
2. Preview image renders using the temporarily generated client-side URL as its `src` attribute

### Issue

- The preview URL is **ephemeral** — only exists during the current browser session
- If the page is refreshed, the image disappears because it's never persisted

### Transitioning Image Handling to the Server

Explored the need for server-side file handling after understanding client-side limitations. The requirement is that users should be able to upload profile images, handled via a form that appears when clicking "edit profile." This bridges the gap between temporary local preview and permanent cloud storage.

---

## 2. Why Not Store Raw Files Directly in Database?

| Reason | Explanation |
|---|---|
| **Performance** | Storing heavy files (images, videos) slows down database operations and increases memory usage |
| **Cost** | Production databases charge for storage — bulk media files increase costs significantly |
| **Delivery / Streaming** | Databases are designed for text data, not optimized for serving or streaming media |
| **Scalability** | Database size grows rapidly with media uploads, making backups and queries slower |

### Solution Pattern
```
File Upload → Multer (server middleware) → Cloudinary (cloud storage) → Returns CDN URL → Store URL in DB
```

---

## 3. Setting Up Multer (Server-Side File Handling)

### What is Multer?
- A Node.js/Express middleware for handling **multipart/form-data**
- Enables uploading of files alongside text fields in a single request
- Built on top of the `busboy` streaming engine

### Installation
```bash
cd server && npm install multer
```

### Creating the Upload Middleware (`uploadMiddleware.js`)

```javascript
const multer = require('multer');

// Memory storage: files kept in RAM as buffers (volatile, fast)
const storage = multer.memoryStorage();

// File filter — only allow image files
const fileFilter = (req, file, cb) => {
  if (file.mimetype.startsWith('image/')) {
    cb(null, true); // accept the file
  } else {
    cb(new Error('Only image files are allowed'), false); // reject
  }
};

// Upload configuration
const upload = multer({
  storage: storage,
  limits: { fileSize: 1024 * 1024 * 1 }, // 1 MB limit
  fileFilter: fileFilter
});

module.exports = upload;
```

### Key Concepts

#### MIME Type (Media Type)
- Indicates the type of content being sent
- Format: `type/subtype` (e.g., `image/jpg`, `video/mp4`)
- Multer's `file.mimetype` is used to validate file types
- Filter checks if mimetype starts with `"image/"`

#### Memory Storage vs.Disk Storage
| Type | Behavior | Use Case |
|---|---|---|
| `memoryStorage()` | Files kept in RAM as Buffer — volatile, fast | Quick processing, then immediate save to cloud/DB |
| `diskStorage()` | Writes temporary copies to disk — persistent | When files need local persistence before processing |

### Using Multer in Routes (`userRoutes.js`)

```javascript
const upload = require('./uploadMiddleware');
const userController = require('./userController');

// "profileImage" is the field key that matches the form input name
router.post('/test-upload', upload.single('profileImage'), userController.testUpload);
```

### Extracting File Data in Controller (`userController.js`)

```javascript
const testUpload = (req, res) => {
  console.log(req.file); // Multer attaches file to the request object
  // req.file contains:
  //   - fieldname: "profileImage"
  //   - originalname: "bruce_profile.jpg"
  //   - encoding: "7bit"
  //   - mimetype: "image/jpeg"
  //   - buffer: <Buffer> — the raw binary data of the file
  res.send('File received');
};

module.exports = { testUpload };
```

---

## 4. Cloudinary Integration (Cloud Media Storage)

### Why Cloudinary?
- Purpose-built cloud platform for managing images, videos, and media
- Returns CDN URLs that can be stored in the database
- Free tier provides ~400 MB of storage — sufficient for personal/student projects
- Supports automatic transformations, optimizations, and processing

### Setup Steps

1. **Create an account** at [cloudinary.com](https://cloudinary.com) (free)

2. **Get API Keys** from Dashboard → Home → Go to API Keys:
   - Cloud Name
   - API Key
   - API Secret

3. **Add credentials** to `.env` file in the server directory:
    ```env
    CLOUD_NAME=your_cloud_name
    API_KEY=your_api_key
    API_SECRET=your_api_secret
    ```

### Upload Flow (to be implemented in next session)
```
1. Multer receives image → creates Buffer
2. Connect to Cloudinary using credentials from .env
3. Stream the Buffer to Cloudinary
4. Cloudinary stores the file and returns a CDN URL
5. Save that URL in the user's profile field in MongoDB
6. Display image using the persistent CDN URL
```

---

## 5. Summary — Complete File Upload Architecture

```
┌─────────────┐        ┌──────────┐        ┌───────────┐        ┌────────┐
│   Client    │──File──▶│  Multer  │──Buf──▶│ Cloudinary│──URL──▶│  DB    │
│  (React)    │ Multipart│(Express) │Data     │(Cloud St.) │Saved  │(MongoDB)│
└─────────────┘ Form Data └──────────┘        └───────────┘       └────────┘
```

---

## Key Takeaways

- **Never** store raw binary files directly in a database (performance, cost, scalability issues)
- Use `URL.createObjectURL()` for client-side preview — it's temporary by design
- **Multer** is the standard middleware for handling file uploads in Express/Node.js
- File validation should always check MIME type to reject non-image files
- Cloud storage services (**Cloudinary, AWS S3, ImgKit**) are designed specifically for media management
- Store only the returned URL/Path in your database — this references the cloud-hosted file

---

## Terms Glossary

| Term | Meaning |
|---|---|
| MIME Type | Media type (e.g., `image/jpeg`) identifying file format |
| Multipart Form Data | HTTP content type for sending text + files together |
| Buffer | Raw binary data representation in Node.js |
| Streaming | Breaking large files into smaller chunks for processing |
| CDN URL | Content Delivery Network URL — publicly accessible web link to a stored asset |
