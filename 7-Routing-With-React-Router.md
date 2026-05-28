# Module 7: Routing with React Router
---

### Core Concept
React Router DOM is the standard routing library for React web applications. It enables client-side navigation between different page views without reloading the page, mapping specific URL paths to corresponding React components.

---

### Syntax and API Quick-Reference
Install the routing package:
```bash
npm install react-router-dom
```

Standard Router Router Configuration Structure (v6):
```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

---

### Production-Ready Example
Here is a complete setup implementing nested layout routing, active links, route parameters (`useParams`), and programmatic redirection (`useNavigate`).

#### Root Configuration: `src/App.jsx`
```jsx
import { BrowserRouter, Routes, Route, NavLink, Link } from 'react-router-dom';
import Home from './pages/Home';
import Profile from './pages/Profile';
import Dashboard from './pages/Dashboard';
import NotFound from './pages/NotFound';

function App() {
  return (
    <BrowserRouter>
      <header className="app-nav">
        {/* NavLink automatically applies an active class based on URL */}
        <NavLink 
          to="/" 
          className={({ isActive }) => isActive ? 'link-active' : 'link'}
        >
          Home
        </NavLink>
        <NavLink 
          to="/dashboard" 
          className={({ isActive }) => isActive ? 'link-active' : 'link'}
        >
          Dashboard
        </NavLink>
      </header>

      <main className="app-content">
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile/:userId" element={<Profile />} />
          <Route path="*" element={<NotFound />} />
        </Routes>
      </main>
    </BrowserRouter>
  );
}

export default App;
```

#### Page Component with Programmatic Navigation: `src/pages/Dashboard.jsx`
```jsx
import { useNavigate } from 'react-router-dom';

function Dashboard() {
  const navigate = useNavigate();

  const handleLogout = () => {
    // Perform authentication cleanups here...
    console.log('Logging user out...');
    
    // Redirect programmatically
    navigate('/');
  };

  return (
    <div className="page">
      <h1>User Dashboard</h1>
      <p>Welcome back! Select a user profile below to view details:</p>
      <ul>
        <li><a href="/profile/1" onClick={(e) => { e.preventDefault(); navigate('/profile/1'); }}>View Profile 1</a></li>
        <li><a href="/profile/2" onClick={(e) => { e.preventDefault(); navigate('/profile/2'); }}>View Profile 2</a></li>
      </ul>
      <button onClick={handleLogout}>Log Out</button>
    </div>
  );
}

export default Dashboard;
```

#### Page Component with Dynamic URL Params: `src/pages/Profile.jsx`
```jsx
import { useParams, useNavigate } from 'react-router-dom';

function Profile() {
  const { userId } = useParams(); // Extracts :userId from current URL
  const navigate = useNavigate();

  return (
    <div className="page">
      <h2>User Profile</h2>
      <p>Displaying data for User ID: <strong>{userId}</strong></p>
      
      <button onClick={() => navigate('/dashboard')}>
        Back to Dashboard
      </button>
    </div>
  );
}

export default Profile;
```

---

### Key Takeaways and Rules
* **No Page Reloads**: Never use standard anchor tags `<a href="/about">` for internal links. These cause the browser to reload the entire app, destroying React state. Always use React Router's `<Link>` or `<NavLink>` components.
* **Catch-All Route**: Map a final Route to `path="*"` to handle any invalid URLs and render a clean custom "404 Not Found" component.
* **Params Extraction**: Route parameters defined with a colon (`path="/profile/:userId"`) can be easily accessed in the matching component using the `useParams()` hook.

---

### Quick Reference Table
| Router Component / Hook | Type | Purpose | Usage Example |
| :--- | :--- | :--- | :--- |
| `<BrowserRouter>` | Parent Wrapper | Provides HTML5 History API context to the application | `<BrowserRouter> ... </BrowserRouter>` |
| `<Routes>` | Switch Board | Groups Route children and returns the best matching route | `<Routes> ... </Routes>` |
| `<Route>` | Configuration | Maps a URL path template to a specific React element | `<Route path="/about" element={<About />} />` |
| `<Link>` | Component | Renders a styled client-side anchor element | `<Link to="/login">Login</Link>` |
| `useParams()` | Custom Hook | Retrieves dynamic params object from active Route | `const { id } = useParams();` |
| `useNavigate()` | Custom Hook | Returns navigate function for programmatic redirects | `const navigate = useNavigate(); navigate('/home');` |

---

### Common Pitfalls and Anti-Patterns
#### 1. Navigating using standard `window.location` or `<a href="...">`
* **Incorrect:**
  ```jsx
  const handleCheckout = () => {
    window.location.href = '/checkout'; // Full browser reload
  };
  ```
* **Why it fails:** This forces the browser to request a new HTML page, clearing all memory, React contexts, and local state variables.
* **Correct:** Use the `useNavigate` hook or `<Link>` component to maintain the client-side single-page app architecture:
  ```jsx
  const navigate = useNavigate();
  const handleCheckout = () => {
    navigate('/checkout'); // Quick SPA transition
  };
  ```

#### 2. Nesting Routes inside Router components without a main path mapping
* **Incorrect:** Placing `<Routes>` inside pages without wrapping the main entry point or placing them outside of `<BrowserRouter>`.
* **Why it fails:** Hooks like `useNavigate()` or `useParams()` will throw a fatal runtime error: *“useNavigate() may be used only in the context of a `<Router>` component.”*
* **Correct:** Always place `<BrowserRouter>` at the highest root level (usually in `main.jsx` or `App.jsx`) enclosing all components that use routing features.
