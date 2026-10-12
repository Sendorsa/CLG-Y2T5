# MERN Project Social Media App: Implementing Posts API for Profile & UI - Complete Revision Notes

## Overview

Building a social media application with authentication context shared across the app, protected routes, and user profile management. This session expanded on registration/login by implementing secured routes and a complete user profile feature, exploring access levels between public and private routes and debugging common routing issues.

## 1. User Registration and Authentication

### Registration and Login Process
- **Registration**: Users create an account; duplicate emails generate conflict errors.
- **Login**: Users log in with email and password. Successful login stores an authentication token.
- **Logout Flow (missing — added)**: On logout, clear the stored token from localStorage/sessionStorage and reset auth state to null so the user is redirected to login/signup pages.

### Protecting Routes with Context API
- **Global State**: Using React's Context API to manage and share global user authentication state across components.
- **AuthProvider**: A provider component that wraps all pages, controls access based on the presence of user data, and globalizes user session information.

#### Implementation Example — AuthContext

```jsx
const AuthContext = createContext(null);

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const navigate = useNavigate();

  useEffect(() => {
    axios.get('/user/me')
      .then(response => setUser(response.data.user))
      .catch(() => setUser(null));
  }, []);

  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
};
```

### Differentiating Public and Protected Routes
- **Public Routes**: Accessible without authentication (e.g., login and sign-up pages).
- **Protected/Private Routes**: Require a valid token; redirect unauthenticated users to login.
- **Issue with Unprotected Routes** (added detail): Logged-out users could still access protected pages directly via URL — resolved by verifying the token before rendering any secured content.

#### Private Route Pattern (missing — added)

```jsx
const PrivateRoute = ({ children }) => {
  const { user } = useContext(AuthContext);
  return user ? children : <Navigate to="/login" />;
};

// Usage:
<PrivateRoute><ProfilePage /></PrivateRoute>
```

#### Auth Hooks — React Hooks Used

| Hook        | Purpose                                              |
|-------------|------------------------------------------------------|
| `useEffect` | Runs on mount to verify user token and update state  |
| `useNavigate` | Redirect users based on authentication status      |
| `useContext`| Consume the AuthContext in nested components         |

```jsx
export const useAuth = () => useContext(AuthContext);
```

## 2. User Profile Management

### Overview
- Profiles are accessed via routes incorporating a username parameter.
- Fetch specific user data using URL parameters to display their profile.
- UI differs depending on whether it is the current user's own profile or another user's.

### Route Parameters

```jsx
import { useParams } from 'react-router-dom';

const ProfilePage = () => {
  const { username } = useParams();
  // Fetch and display user data based on username via useAuth / API
};
```

### Component Layout (additive detail)

- **ProfileHeader**: Avatar, banner, name/bio, action buttons.
- **ProfileTabs**: Posts, Media, Likes tabs.
- **UserPostsGrid**: Grid/list layout of posts.
- **Edit Profile Modal/Draft**: For self-view editing of bio/avatar.

### Conditional Rendering

| View        | Shown To        | Key UI Elements                    |
|-------------|-----------------|------------------------------------|
| Self Profile| Current user    | Edit avatar, edit bio, save/cancel |
| Other User  | Anyone else     | Follow / Unfollow / Message button |

## 3. Debugging and Issues Encountered

- **Direct URL Access**: Users could navigate to protected routes directly through the browser — resolved by setting up proper loading states in Context before rendering secured content.
- **Auth Race Condition (added)**: On initial page load, the token might not have been verified yet, causing brief flashes of auth-dependent UI. Solved by introducing a `loading` state in AuthContext and guarding all views until the state resolves.
- **Profile Data Fetching**: Ensured consistent data flow between frontend and backend by updating frontend logic to properly handle dynamic user profile routes and stale cache scenarios.

## 4. What Comes Next

- Complete the AuthContext implementation (add login, register, logout, `loading` state).
- Connect Posts API to profile page and other UI components.
- Add follow/unfollow functionality for other-user profiles.
- Implement route-level code splitting and lazy-load heavy pages (e.g., ProfilePage) for better performance.

## 5. Glossary

| Term | Definition |
|------|-----------|
| Auth Provider | Component to wrap pages and provide authentication state |
| Axios | Promise-based HTTP client for the browser and Node.js |
| Context | Shared data structure in React for global information |
| Login | Authenticating a user with email and password |
| Protected Page | A page that requires user authentication to access |
| Protected Route | Routes that require authentication to access |
| Public Route | Routes accessible without authentication |
| Registration | Process of creating a user account in the system |
| Token | A unique identifier used to authenticate a user's session |
| Use Effect | React hook that runs on every render or updates |
| Use Params | React Router hook to access URL parameters |
| User Data | Information about the currently authenticated user |
