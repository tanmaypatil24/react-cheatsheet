# Module 5: useEffect and Lifecycle
---

### Core Concept
The `useEffect` hook enables functional components to perform side-effects, such as data fetching, subscription management, manual DOM manipulation, and setting up timers. It serves as a unified API representing component lifecycle events (mount, update, and unmount).

---

### Syntax and API Quick-Reference
```jsx
import { useEffect } from 'react';

useEffect(() => {
  // Side-effect logic goes here

  return () => {
    // Optional cleanup function (runs before re-running effect or unmounting)
  };
}, [dependencyArray]);
```

#### Dependency Array Behaviors:
1. **No Dependency Array**: `useEffect(() => {})`
   * Runs on every single render (mount + every update).
2. **Empty Dependency Array**: `useEffect(() => {}, [])`
   * Runs exactly once (when the component mounts).
3. **With Dependencies**: `useEffect(() => {}, [val1, val2])`
   * Runs once on mount, and then re-runs only when the specified variables change.

---

### Production-Ready Example
Here is a data fetching component that cleanly handles network requests, loading states, error states, and cleanups using `AbortController` to prevent memory leaks and race conditions.

#### Component: `src/components/UserFetcher.jsx`
```jsx
import { useState, useEffect } from 'react';

function UserFetcher() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    const { signal } = controller;

    const fetchUsers = async () => {
      try {
        setLoading(true);
        const response = await fetch('https://jsonplaceholder.typicode.com/users', { signal });
        
        if (!response.ok) {
          throw new Error('Failed to fetch user directory');
        }
        
        const data = await response.json();
        setUsers(data);
        setError(null);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();

    // Cleanup: Abort fetch when component unmounts or effect re-runs
    return () => {
      controller.abort();
    };
  }, []); // Empty dependency array: runs on mount only

  if (loading) return <p>Loading system users...</p>;
  if (error) return <p className="error-text">Error: {error}</p>;

  return (
    <div className="fetcher-container">
      <h2>User Directory</h2>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            <strong>{user.name}</strong> - {user.email}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default UserFetcher;
```

---

### Key Takeaways and Rules
* **Syncing External Systems**: Use `useEffect` only to synchronize your components with non-React elements (such as APIs, browser events, or window dimensions). Do not use it for calculations derived directly from props or state.
* **Cleanup Function Purpose**: The function returned from `useEffect` runs before the component is unmounted and before every subsequent run of the effect. Use it to cancel subscriptions, clean up event listeners, abort fetches, and clear timers.
* **Don't Forget Dependencies**: Every variable from the component scope (props, state, or derived variables) read inside `useEffect` MUST be declared in the dependency array. Failing to do so causes "stale closure" bugs.

---

### Quick Reference Table
| Lifecycle Event | Equivalent `useEffect` Configuration | Setup Example |
| :--- | :--- | :--- |
| **Mount** (Insertion in DOM) | Empty dependency array | `useEffect(() => {}, [])` |
| **Update** (Prop/State changes) | With specific items in dependency array | `useEffect(() => {}, [stateVar])` |
| **Unmount** (Removal from DOM) | Returned cleanup function inside empty effect | `useEffect(() => { return () => cleanup(); }, [])` |

---

### Common Pitfalls and Anti-Patterns
#### 1. Infinite rendering loop due to missing dependencies
* **Incorrect:**
  ```jsx
  const [data, setData] = useState([]);
  useEffect(() => {
    fetch('api/url')
      .then(res => res.json())
      .then(result => setData(result)); // Setting state triggers re-render
  }); // No dependency array
  ```
* **Why it fails:** The effect sets state (`setData`), which triggers a re-render. Since there is no dependency array, the effect runs again, triggers another state update, and loops infinitely.
* **Correct:** Provide an empty dependency array `[]` (if running only on mount) or add conditions to prevent repetitive triggers:
  ```jsx
  useEffect(() => {
    fetch('api/url')
      .then(res => res.json())
      .then(result => setData(result));
  }, []); // Runs once on mount
  ```

#### 2. Neglecting to clean up event listeners or timers
* **Incorrect:**
  ```jsx
  useEffect(() => {
    window.addEventListener('resize', handleResize);
  }, []);
  ```
* **Why it fails:** When the component unmounts, the event listener is still attached to the global `window` object. This causes a memory leak and potential app crashes when the handler references unmounted components.
* **Correct:** Return a cleanup function that removes the listener:
  ```jsx
  useEffect(() => {
    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);
  ```
