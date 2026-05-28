# Module 4: Conditional Rendering and Lists
---

### Core Concept
Conditional rendering allows you to show or hide elements in your UI based on certain conditions or states. List rendering lets you loop through arrays of data and dynamically generate JSX components using the JavaScript `.map()` method.

---

### Syntax and API Quick-Reference
Conditional Rendering using Ternary Operator:
```jsx
{condition ? <ActiveComponent /> : <InactiveComponent />}
```

Conditional Rendering using Short-circuit evaluation (`&&`):
```jsx
{isLoggedIn && <Dashboard />}
```

List Rendering with unique `key` prop:
```jsx
<ul>
  {items.map((item) => (
    <li key={item.id}>{item.name}</li>
  ))}
</ul>
```

---

### Production-Ready Example
Let's create a functional task list component that displays list items based on completeness and handles empty states gracefully.

#### Component: `src/components/TaskManager.jsx`
```jsx
import { useState } from 'react';

function TaskManager() {
  const [tasks] = useState([
    { id: 1, text: 'Learn React Setup', completed: true },
    { id: 2, text: 'Master JSX & Props', completed: true },
    { id: 3, text: 'Understand State & Events', completed: false },
    { id: 4, text: 'Build a Conditional List App', completed: false }
  ]);

  const [showCompletedOnly, setShowCompletedOnly] = useState(false);

  // Filter tasks based on condition
  const filteredTasks = showCompletedOnly 
    ? tasks.filter(task => task.completed) 
    : tasks;

  return (
    <div className="task-manager">
      <h2>Task Manager</h2>
      
      <button onClick={() => setShowCompletedOnly(prev => !prev)}>
        {showCompletedOnly ? "Show All Tasks" : "Show Completed Only"}
      </button>

      <h3>Task List</h3>

      {filteredTasks.length === 0 ? (
        <p className="no-tasks-msg">No tasks found match the criteria.</p>
      ) : (
        <ul className="task-list">
          {filteredTasks.map((task) => (
            <li 
              key={task.id} 
              className={task.completed ? 'task-complete' : 'task-pending'}
            >
              <span>{task.text}</span>
              <span>{task.completed ? " (Done)" : " (Pending)"}</span>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

export default TaskManager;
```

---

### Key Takeaways and Rules
* **Ternary vs &&**: Use ternaries (`condition ? A : B`) when you have an "if-else" scenario. Use `&&` short-circuiting when you want to render something only if a condition is true, and nothing if it is false.
* **Why Key Prop is Essential**: React uses the `key` prop during reconciliation to identify which items have changed, been added, or been removed. It is critical for performance and DOM state retention.
* **Keys Must Be Stable and Unique**: A key must be unique among sibling elements. Do not generate keys on the fly (e.g., `key={Math.random()}`).
* **Empty State Handling**: Always plan for empty list arrays and render alternative text or fallback components instead of an empty screen.

---

### Quick Reference Table
| Rendering Pattern | Syntax / Usage | Ideal Use Case |
| :--- | :--- | :--- |
| If / Else Block | `if (cond) { return A; } else { return B; }` | Returning entirely different UI trees from a component |
| Ternary Operator | `condition ? <A /> : <B />` | Inline conditional selection between two options |
| Logical AND | `condition && <A />` | Inline conditional display of a single element |
| Map Loop | `array.map(item => <Item key={item.id} />)` | Rendering data arrays dynamically |

---

### Common Pitfalls and Anti-Patterns
#### 1. Using array indexes as list keys
* **Incorrect:**
  ```jsx
  items.map((item, index) => (
    <li key={index}>{item.name}</li>
  ));
  ```
* **Why it fails:** If items are reordered, deleted, or inserted at the beginning, the indices change. This causes React to mismatch DOM elements, leading to bugs in input states, animations, or styling.
* **Correct:** Use a unique, persistent identifier (like a database ID or GUID):
  ```jsx
  items.map((item) => (
    <li key={item.id}>{item.name}</li>
  ));
  ```

#### 2. Rendering numbers that resolve to `0` with the `&&` operator
* **Incorrect:**
  ```jsx
  const [unreadCount, setUnreadCount] = useState(0);
  return (
    <div>
      {unreadCount && <p>You have new messages</p>}
    </div>
  );
  ```
* **Why it fails:** In JavaScript, `0 && expression` evaluates to `0`. React renders `0` directly into the DOM instead of leaving the layout empty.
* **Correct:** Ensure the condition resolves to a boolean value:
  ```jsx
  {unreadCount > 0 && <p>You have new messages</p>}
  ```