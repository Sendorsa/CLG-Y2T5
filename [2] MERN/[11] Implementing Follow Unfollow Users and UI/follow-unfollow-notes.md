# MERN Social Media App — Follow/Unfollow Users

## 1. Authentication & Profile Bug Fixes


- **Issue**: Username route parameter wasn't being passed correctly to the server, causing profile page redirects and slow loading.
- **Fix**: Corrected the username extraction from route params so it's properly sent to the server for lookup.

---

## 2. Dynamic Profile Page by Username


### How It Works

1. User enters a username on the profile page → triggers a search.
2. The frontend sends the username to the backend via `GET /users/profile/:username`.
3. Backend searches the database and returns the user's profile data (`profileData`: name, pass, email, etc.).
4. Frontend receives the data and re-renders the profile accordingly.

### Implementation

- **Both a function + `useEffect`** is used (best architecture):
  - An **async function** calls `axios.get('/users/profile/username')`.
  - **`useEffect`** watches for changes in the `username` route parameter and re-runs the function whenever it changes.

```js
// Inside Profile component
const getProfile = async () => {
  const res = await axios.get(`/users/profile/${username}`);
  setUserData(res.data.profileData);
};

useEffect(() => {
  getProfile();
}, [username]);
```

### Handling Response Data

- Success response structure from transcript: `{ message: 'User found', data: { profileData: { name, email, ... } } }` — accessed as `res.data.profileData` in notes. Actual field names depend on server implementation.
- Error handling should be added for cases where the user doesn't exist.

---

## 3. Distinguishing Own Profile vs. Other Users' Profiles


There are **two types of profiles** to handle:

| Type | Source of Data |
|------|---------------|
| **Own profile** | Already available via `useUser` hook (`loggedInUser`) |
| **Other user's profile** | Fetched dynamically from server via API |

### Check if viewing own profile:

```js
const isOwnProfile = loggedInUser.username === username;
```

- If **same** → use data from `useUser`.
- If **different** → use data fetched from the profile API (`userData` state).

---

## 4. Glossary of Terms


| Term | Definition |
|------|------------|
| **Authentication Middleware** | A component that checks user credentials and permissions before allowing access to certain parts of an application. |
| **Controller** | A function that receives HTTP requests, processes them, and sends back a response. |
| **getUserProfile Controller** | A controller that retrieves the profile data of a user based on their username. |
| **Routing** | The process of selecting which controller should respond to a request in a web app. |
| **useEffect Hook** | A React Hook that lets you perform side effects in function components. |
| **useState Hook** | A React Hook that lets you add state to function components. |
| **Async Function** | A JavaScript function that returns a Promise and can use the `await` keyword within it to get results from asynchronous operations. |
| **try-catch Block** | A construct in JavaScript used to handle exceptions and errors during code execution. |
| **Template Literals** | A JavaScript syntax that allows for embedding variables and expressions within a string using backticks. |
| **isOwnProfile Variable** | A boolean variable used to determine if the profile being viewed is the logged-in user's profile. |
| **Conditional Rendering** | A technique in React to render different output based on conditional logic. |
| **addToSet Operator** | A MongoDB/Mongoose operator that adds a value to an array only if it does not already exist in the set. |

---

## 5. Schema Setup for Followers & Following Relationships


### Mongoose Schema — ObjectId References

The `followers` and `following` arrays store **ObjectIds** that reference users:

```js
followers: {
  type: [mongoose.Schema.Types.ObjectId],
  ref: 'User'
}
```

- **Why `ref: 'User'`?** So Mongoose knows these ObjectIds point to documents in the `users` collection and can be population-ready.
- **Why `{ type: [..., ref] }` instead of just an array?** Because ObjectIds are objects with two fields — `type` and `ref`. Without this, Mongoose would store plain IDs without reference context.

---

## 6. Follow User Controller — `followUser`


### Route
```
PUT /api/users/profile/follow/:id
```
- `:id` = target user's MongoDB ObjectId (sent from client).
- Current user comes from authentication middleware (`req.user`).

### Step-by-step Logic

1. **Extract IDs**:
   - `currentUserId` → from authenticated token/middleware.
   - `targetUserId` → from route params.

2. **Validation — Cannot follow yourself**:
   ```js
   if (currentUserId.toString() === targetUserId) {
     return res.status(409).json({ message: "You cannot follow yourself" });
   }
   ```

3. **Validation — Already following** (check before updating):
   ```js
   const targetUser = await User.findById(targetUserId);
   const alreadyFollowing = targetUser.followers.some(
     follower => follower.toString() === currentUserId
   );
   if (alreadyFollowing) {
     return res.status(409).json({ message: "User already following" });
   }
   ```

4. **Update using Mongoose operators**:
   ```js
   // Add targetUserId to current user's followings
   await User.findByIdAndUpdate(currentUserId, {
     $addToSet: { following: targetUserId }
   });

   // Add currentUserId to target user's followers
   await User.findByIdAndUpdate(targetUserId, {
     $addToSet: { followers: currentUserId }
   });
   ```

5. **Response**: Return confirmed target user data.

---

## 7. Unfollow User Controller — `unfollowUser`


### Route
```
PUT /api/users/profile/unfollow/:id
```

### Logic (mirrors follow)

1. Extract `currentUserId` and `targetUserId`.
2. **Validation — Cannot unfollow yourself**:
   ```js
   if (currentUserId.toString() === targetUserId) {
     return res.status(409).json({ message: "You cannot unfollow yourself" });
   }
   ```
3. **Remove using `$pull` operator**:
   ```js
   // Remove current user from target's followers
   await User.findByIdAndUpdate(targetUserId, {
     $pull: { followers: currentUserId }
   });

   // Remove target from current user's followings
   await User.findByIdAndUpdate(currentUserId, {
     $pull: { following: targetUserId }
   });
   ```
4. **Response**: Return confirmation message (e.g., "User unfollowed").

---

## 8. Mongoose Operators Used


| Operator | Purpose | Example Use |
|----------|---------|-------------|
| `$addToSet` | Add value to array if not already present | Adding user IDs to `followers`/`following` arrays |
| `$pull` | Remove all instances of a value from an array | Removing user IDs when unfollowing |

### Why these instead of standard methods?

- **$addToSet** prevents duplicate entries automatically.
- **$pull** cleanly removes references without manual array manipulation.

---

## 9. Client-Side Integration


### What the client needs to send:

| Needed | Source |
|--------|--------|
| Target user ID (`_id`) | From `userData` state (fetched profile data) |
| Current user ID | Available from authentication context/hook |

### Follow/Unfollow Button Flow:

1. Click "Follow" button → pick target user's `_id`.
2. Send it via route parameter to follow controller.
3. Follow/unfollow logic executes on server (updates both users' arrays).
4. Client refreshes or optimistically updates UI state.

---

## 10. Key Takeaways


- Both **function + `useEffect`** is the proper React pattern for API calls triggered by changing parameters.
- Server-side validations are critical — don't trust client-side checks alone.
- Use Mongoose `$addToSet` and `$pull` operators for atomic array updates in relationships.
- Always define `ref: 'User'` on ObjectId fields that reference other collections so Mongoose can handle references properly.

---

## 11. Profile Link from Home Page


- A "Profile" link is added to the home page for direct navigation to the logged-in user's profile.
- If username changes, the profile should automatically reflect the new user's data.
- Users can search other usernames on the profile page, which triggers a server request and displays that user's public information.

---

## 12. UI Conditional Rendering on Profile Page


The profile page renders different buttons based on whether the viewer is observing their own profile or another user's:

```jsx
{isOwnProfile ? (
  <button>Edit Profile</button>
) : (
  <button 
    onClick={handleFollow}
  >
    {isFollowing ? 'Unfollow' : 'Follow'}
  </button>
)}
```

- **`isOwnProfile`**: Determined by comparing `loggedInUser.username === username`.
- **`isFollowing`**: State that tracks whether the logged-in user is currently following the target user. Fetch from `userData.followers` or maintain in local state after follow/unfollow API calls.

### Button States When Viewing Another User:

| Condition | Button Text | Action on Click |
|-----------|-------------|-----------------|
| Not following | "Follow" | POST to `/users/follow/:id` / PUT `/api/users/profile/follow/:id` |
| Already following | "Unfollow" | POST to `/users/unfollow/:id` / PUT `/api/users/profile/unfollow/:id` |

---

## 13. Follow State Management on Client


After a follow/unfollow action, the UI must stay in sync:

```js
const handleFollow = async () => {
  try {
    if (isFollowing) {
      await axiosInstance.put(`/api/users/profile/unfollow/${targetUserId}`);
    } else {
      await axiosInstance.put(`/api/users/profile/follow/${targetUserId}`);
    }
    setIsFollowing(!isFollowing);
    // Optionally update userData.followers count
    setUserData(prev => ({
      ...prev,
      followersCount: isFollowing 
        ? prev.followersCount - 1 
        : prev.followersCount + 1
    }));
  } catch (error) {
    console.error('Follow/Unfollow error:', error);
  }
};
```

- **Optimistic updates** update UI instantly before server response for better UX.
- **Fallback**: If server call fails, revert state to match actual data.

---

## 14. Login Flow on Profile Page


- When a **non-logged-in user** visits the profile page, show a "Login" button instead of Follow/Edit Profile.
- After login, redirect back to the original profile URL (use return URL parameter).
- This ensures seamless navigation: view another user's profile → click Login → return to that profile page.

---

## 15. Analogies (For Understanding)


| Concept | Analogy |
|---------|---------|
| Logged-in vs Viewed User | Like a movie streaming platform — the homepage shows personalized recommendations while you can explore other users' watchlists |
| `$addToSet` | Like adding a name to a group chat — if already in, it's ignored; if not, they're added |
| `$pull` | Like removing someone from the group chat entirely |

---

## 16. Next Steps (From Lecture)


- Connect follow/unfollow buttons on the client side to the controllers.
- **Edit Profile** functionality — handling image uploads server-side.
- Proper media/image processing on the backend.

---

*Notes compiled from lecture: MERN-10 Project — Build a Social Media App (Implementing Follow/Unfollow Users)*
