# Module 13: Custom Hooks
---

### Core Concept
Custom hooks are JavaScript functions that start with the prefix `use` and can call other React hooks. They allow you to extract component logic into reusable functions, separating UI markup from stateful behavior. This makes code dry (Don't Repeat Yourself), modular, and highly testable.

---

### Syntax and API Quick-Reference
General structure of a custom hook:
```jsx
import { useState, useEffect } from 'react';

function useMyCustomHook(param) {
  const [data, setData] = useState(null);

  useEffect(() => {
    // Custom hook side-effects...
  }, [param]);

  return [data, setData]; // Can return values, arrays, or objects
}
```

---

### Production-Ready Example
Let's build two robust, production-grade custom hooks: `useLocalStorage` and `useFetch`.

#### Hook 1: `src/hooks/useLocalStorage.js` (State sync with localStorage)
```jsx
import { useState, useEffect } from 'react';

export function useLocalStorage(key, initialValue) {
  // Read value from local storage once on initialization
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  // Keep localStorage synchronized whenever state changes
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  }, [key, storedValue]);

  return [storedValue, setStoredValue];
}
```

#### Hook 2: `src/hooks/useFetch.js` (Network states with Abort Controller)
```jsx
import { useState, useEffect } from 'react';

export function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!url) return;

    const controller = new AbortController();
    const { signal } = controller;

    const fetchData = async () => {
      setLoading(true);
      try {
        const response = await fetch(url, { signal });
        if (!response.ok) {
          throw new Error(`Fetch error: ${response.statusText}`);
        }
        const json = await response.json();
        setData(json);
        setError(null);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchData();

    return () => {
      controller.abort(); // Cancel request if component unmounts
    };
  }, [url]);

  return { data, loading, error };
}
```

#### Consumption Component: `src/components/UserDashboard.jsx`
```jsx
import { useLocalStorage } from '../hooks/useLocalStorage';
import { useFetch } from '../hooks/useFetch';

function UserDashboard() {
  const [token, setToken] = useLocalStorage('auth_token', '');
  const { data: users, loading, error } = useFetch('https://jsonplaceholder.typicode.com/users');

  return (
    <div className="dashboard">
      <h2>System Dashboard</h2>
      
      <section className="token-section">
        <label>Session Token:</label>
        <input 
          type="text" 
          value={token} 
          onChange={(e) => setToken(e.target.value)} 
          placeholder="Enter credentials..."
        />
      </section>

      <section className="users-section">
        <h3>User List</h3>
        {loading && <p>Loading directory...</p>}
        {error && <p className="error">Error loading users: {error}</p>}
        {users && (
          <ul>
            {users.map(u => <li key={u.id}>{u.name} ({u.email})</li>)}
          </ul>
        )}
      </section>
    </div>
  );
}

export default UserDashboard;
```

---

### Key Takeaways and Rules
* **Strict "use" Naming Prefix**: Custom hooks must always start with `use` (e.g. `useFetch`, `useAuth`). This naming convention tells React's linter to enforce the standard Rules of Hooks on the function.
* **Isolation of State**: Every time a component calls a custom hook, the state and effects inside the hook are completely isolated. Multiple components consuming the same custom hook do not share state; they share the stateful *logic*.
* **Rules of Hooks Apply**: Custom hooks must follow the same rules as built-in hooks: they can only be called at the top level of your component or inside other hooks, never inside loops, conditions, or nested functions.

---

### Quick Reference Table
| Custom Hook Idea | Expected Return | Ideal Use Case |
| :--- | :--- | :--- |
| `useFetch(url)` | `{ data, loading, error }` | Centralizing and caching server API communications |
| `useLocalStorage(key, init)` | `[state, setState]` | Automatic persistent browser states (themes, forms, tokens) |
| `useWindowSize()` | `{ width, height }` | Listening to window resizing events for responsive JavaScript layouts |
| `useAuth()` | `{ user, login, logout }` | Managing authentication contexts and credentials easily |

---

### Common Pitfalls and Anti-Patterns
#### 1. Calling custom hooks conditionally
* **Incorrect:**
  ```jsx
  function Profile({ isUserLoggedIn }) {
    if (isUserLoggedIn) {
      const data = useFetch('/profile-data'); // Hook called conditionally
    }
    return <div>Profile Details</div>;
  }
  ```
* **Why it fails:** This violates the fundamental Rule of Hooks. React relies on the call order of hooks to remain identical across every render. Calling hooks inside `if` statements shifts the order, leading to internal React state corruption.
* **Correct:** Always call the hook at the top level. Move the conditional evaluation inside the hook or pass conditional parameters instead:
  ```jsx
  function Profile({ isUserLoggedIn }) {
    // Hook called at top level; handles condition internally (only fetches if url is non-null)
    const { data } = useFetch(isUserLoggedIn ? '/profile-data' : null);
    
    return <div>Profile Details</div>;
  }
  ```

#### 2. Believing custom hooks share global state naturally
* **Incorrect:** Assuming that if Component A calls `useLocalStorage('theme')` and Component B calls `useLocalStorage('theme')`, changing state in Component A will automatically trigger a state re-render in Component B.
* **Why it fails:** Custom hooks do not share state instances. Each hook invocation sets up its own separate reactive variables.
* **Correct:** If real-time state sharing across separate components is required, wrap the custom hook consumption inside a React Context Provider (Module 10) or utilize global state stores like Zustand or Redux.
