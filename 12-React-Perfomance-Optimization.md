# Module 12: React Performance Optimization
---

### Core Concept
React is highly performant by default. However, in large applications, unnecessary re-renders can degrade performance. React provides three primary optimization APIs: `React.memo` (to skip re-rendering unchanged components), `useMemo` (to cache expensive computations), and `useCallback` (to cache function definitions across renders).

---

### Syntax and API Quick-Reference
Caching function definitions:
```jsx
import { useCallback } from 'react';

const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]); // Re-creates function ONLY if 'a' or 'b' changes
```

Caching calculated values:
```jsx
import { useMemo } from 'react';

const memoizedValue = useMemo(() => {
  return calculateExpensiveValue(a, b);
}, [a, b]); // Re-runs computation ONLY if 'a' or 'b' changes
```

Memoizing functional components:
```jsx
import React from 'react';

const MemoizedComponent = React.memo(MyComponent);
```

---

### Production-Ready Example
Here is a dashboard setup implementing all three optimization strategies to prevent rendering lag.

#### Optimized Child Component: `src/components/ListItem.jsx`
```jsx
import React from 'react';

// Prevents re-rendering unless 'item' or 'onRemove' references change
const ListItem = React.memo(function ListItem({ item, onRemove }) {
  console.log(`Rendering ListItem: ${item.name}`);
  return (
    <div className="list-item">
      <span>{item.name}</span>
      <button onClick={() => onRemove(item.id)}>Remove</button>
    </div>
  );
});

export default ListItem;
```

#### Main Dashboard: `src/components/AdminDashboard.jsx`
```jsx
import { useState, useMemo, useCallback } from 'react';
import ListItem from './ListItem';

function AdminDashboard() {
  const [items, setItems] = useState([
    { id: 1, name: 'Tanmay', score: 95 },
    { id: 2, name: 'Amit', score: 85 },
    { id: 3, name: 'Sneha', score: 98 }
  ]);
  const [theme, setTheme] = useState('light');

  // Optimization 1: useCallback prevents recreation of onRemove on every render.
  // This keeps ListItem (which is wrapped in React.memo) from re-rendering when theme changes.
  const handleRemove = useCallback((id) => {
    setItems((prevItems) => prevItems.filter(item => item.id !== id));
  }, []); // Empty dependencies: function reference never changes

  // Optimization 2: useMemo caches the sorting calculation.
  // It only re-calculates when the 'items' array changes, not when 'theme' changes.
  const sortedHighScores = useMemo(() => {
    console.log('Computing expensive sort operation...');
    return [...items].sort((a, b) => b.score - a.score);
  }, [items]);

  return (
    <div className={`dashboard ${theme}`}>
      <h2>Dashboard Performance Tracker</h2>
      <button onClick={() => setTheme(prev => prev === 'light' ? 'dark' : 'light')}>
        Toggle Theme (Current: {theme})
      </button>

      <h3>High Scores</h3>
      <div className="list">
        {sortedHighScores.map(item => (
          <ListItem 
            key={item.id} 
            item={item} 
            onRemove={handleRemove} 
          />
        ))}
      </div>
    </div>
  );
}

export default AdminDashboard;
```

---

### Key Takeaways and Rules
* **Avoid Premature Optimization**: Do not wrap every single component in `React.memo` or use `useMemo` for basic calculations. Optimization adds complexity and dependency array comparison overhead.
* **When to use `React.memo`**: Apply it to components that have many children, re-render often, and receive stable, unchanging props.
* **Referential Integrity**: Passing arrays `[]`, objects `{}`, or inline functions `() => {}` as props to a child component will completely break `React.memo` because a new reference is created on every render. Use `useMemo` and `useCallback` on the parent to pass stable references.

---

### Quick Reference Table
| Optimization API | Caches | Used For | Dependency Dependent |
| :--- | :--- | :--- | :--- |
| `React.memo(Component)` | Component DOM Output | Skipping re-renders of a child component when props don't change | Yes (does shallow comparison of props) |
| `useMemo(() => value, deps)` | Value / Object reference | Skipping expensive CPU calculations or maintaining object referential integrity | Yes |
| `useCallback(fn, deps)` | Function definition / reference | Preventing recreation of callback functions passed down to optimized children | Yes |

---

### Common Pitfalls and Anti-Patterns
#### 1. Using `useCallback` on simple handlers with un-memoized children
* **Incorrect:**
  ```jsx
  // Parent
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);

  return <Button onClick={handleClick} /> // Button is a simple unmemoized component
  ```
* **Why it fails:** Since `<Button>` is not wrapped in `React.memo`, it will re-render whenever the parent renders regardless of whether `handleClick` has a stable reference. The use of `useCallback` here is redundant and adds unnecessary CPU overhead.
* **Correct:** Use standard functions for regular child components; use `useCallback` only when passing functions to optimized child components or inside dependencies of other hooks.

#### 2. Writing empty dependency arrays on functions that read component state
* **Incorrect:**
  ```jsx
  const [text, setText] = useState('');
  
  // Bug: 'text' is missing from the dependency array
  const handleAlert = useCallback(() => {
    alert(text); 
  }, []); 
  ```
* **Why it fails:** The callback is memoized on the first render, capturing a "stale closure" of `text` as an empty string `""`. Even if the user types into the input and updates `text`, triggering the handler will always alert an empty string.
* **Correct:** Always declare all reactive variables used inside the callback in the dependency array:
  ```jsx
  const handleAlert = useCallback(() => {
    alert(text);
  }, [text]); // Re-creates function definition when text updates
  ```
