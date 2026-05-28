# Module 3: State and Events
---

### Core Concept
State represents the dynamic data stored inside a React component that can change over time. When state changes, React automatically re-renders the component to reflect the new state. Events are user interactions (like clicks, keypresses, or form submissions) that trigger handlers to update this state.

---

### Syntax and API Quick-Reference
The `useState` hook is used to declare state variables:
```jsx
import { useState } from 'react';

const [state, setState] = useState(initialValue);
```
* `state`: The current value of the state.
* `setState`: A function to update the state value and trigger a re-render.
* `initialValue`: The starting state value (can be a number, string, boolean, array, or object).

---

### Production-Ready Example
Here is a complete component showing state updates (with dynamic additions and subtractions) alongside controlled input mirroring.

#### Component: `src/components/InteractivePanel.jsx`
```jsx
import { useState } from 'react';

function InteractivePanel() {
  const [count, setCount] = useState(0);
  const [inputText, setInputText] = useState('');

  const handleIncrement = () => setCount((prevCount) => prevCount + 1);
  const handleDecrement = () => setCount((prevCount) => prevCount - 1);
  const handleReset = () => setCount(0);

  const handleInputChange = (event) => {
    setInputText(event.target.value);
  };

  return (
    <div className="interactive-panel">
      <section className="counter-section">
        <h2>Counter: {count}</h2>
        <button onClick={handleIncrement}>Increment</button>
        <button onClick={handleDecrement}>Decrement</button>
        <button onClick={handleReset}>Reset</button>
      </section>

      <hr />

      <section className="input-section">
        <h3>Input Mirror</h3>
        <input 
          type="text" 
          placeholder="Type something..." 
          value={inputText} 
          onChange={handleInputChange} 
        />
        <p>Live Output: <strong>{inputText || '(Empty)'}</strong></p>
      </section>
    </div>
  );
}

export default InteractivePanel;
```

---

### Key Takeaways and Rules
* **Never Mutate State Directly**: Always use the state setter function (e.g., `setState`). Directly modifying state variables (e.g., `count = count + 1`) will not trigger a re-render.
* **State Updates are Asynchronous**: React batches state updates for performance. Reading state immediately after setting it will yield the old value.
* **Functional Updates**: When updating state based on its previous value, always pass a callback function to the state setter (e.g., `setCount(prev => prev + 1)`). This avoids stale state bugs.
* **Controlled Inputs**: Binding an input's `value` to a state variable and updating it via `onChange` makes it a "controlled component."

---

### Quick Reference Table
| Event / Function | Trigger Mechanism | Common React Event Prop |
| :--- | :--- | :--- |
| `useState(init)` | Hook initialization | Called once inside component setup |
| Click event | Mouse click on elements | `onClick={handler}` |
| Input change | Value changes in inputs/textareas | `onChange={handler}` |
| Form submission | Form is submitted | `onSubmit={handler}` |
| Focus / Blur | Input gains or loses focus | `onFocus={handler}` / `onBlur={handler}` |

---

### Common Pitfalls and Anti-Patterns
#### 1. Mutating state objects or arrays directly
* **Incorrect:**
  ```jsx
  const [user, setUser] = useState({ name: 'Tanmay', age: 24 });
  const birthday = () => {
    user.age = 25; // Direct mutation
    setUser(user); 
  };
  ```
* **Why it fails:** React performs shallow reference comparisons. Since the reference of the `user` object remains the same, React may skip re-rendering.
* **Correct:** Copy the object using the spread operator (`...`) to create a new reference:
  ```jsx
  const birthday = () => {
    setUser(prevUser => ({
      ...prevUser,
      age: 25
    }));
  };
  ```

#### 2. Reading state immediately after an update
* **Incorrect:**
  ```jsx
  const [score, setScore] = useState(0);
  const addScore = () => {
    setScore(score + 1);
    console.log(score); // Will print 0 instead of 1
  };
  ```
* **Why it fails:** State updates are scheduled and async. The `score` variable inside the current scope is constant during this render lifecycle.
* **Correct:** Use a variable to capture the new value, or use a `useEffect` hook to run side-effects on state change:
  ```jsx
  const addScore = () => {
    const nextScore = score + 1;
    setScore(nextScore);
    console.log(nextScore); // Safely prints the new value
  };
  ```