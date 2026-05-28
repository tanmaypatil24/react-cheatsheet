# Module 9: Basic Mini Project
---

### Core Concept
Putting it all together: Building a functional, state-driven application. We will implement a complete, robust To-Do App with task filters, local storage persistence, item counters, completion toggles, and deletion.

---

### Syntax and API Quick-Reference
Key state operations used:
* **Add**: `setTasks(prev => [...prev, newTask])`
* **Toggle Complete**: `setTasks(prev => prev.map(t => t.id === id ? { ...t, completed: !t.completed } : t))`
* **Delete**: `setTasks(prev => prev.filter(t => t.id !== id))`
* **Local Storage Persistence**:
  * Save: `localStorage.setItem('tasks', JSON.stringify(tasks))`
  * Read: `JSON.parse(localStorage.getItem('tasks')) || []`

---

### Production-Ready Example
Here is the complete source code for a modular, clean, and interactive To-Do Application.

#### Component 1: `src/components/TaskForm.jsx` (Handles adding new items)
```jsx
import { useState } from 'react';

function TaskForm({ onAddTask }) {
  const [text, setText] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!text.trim()) return;
    onAddTask(text);
    setText(''); // Reset input
  };

  return (
    <form onSubmit={handleSubmit} className="task-form">
      <input
        type="text"
        placeholder="Add a new task..."
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button type="submit">Add Task</button>
    </form>
  );
}

export default TaskForm;
```

#### Component 2: `src/components/TaskItem.jsx` (Renders individual tasks)
```jsx
function TaskItem({ task, onToggleComplete, onDeleteTask }) {
  return (
    <div className={`task-item ${task.completed ? 'completed' : ''}`}>
      <input
        type="checkbox"
        checked={task.completed}
        onChange={() => onToggleComplete(task.id)}
      />
      <span className="task-text">{task.text}</span>
      <button 
        onClick={() => onDeleteTask(task.id)} 
        className="delete-btn"
      >
        Delete
      </button>
    </div>
  );
}

export default TaskItem;
```

#### Main Application: `src/App.jsx` (Lifts state and integrates sub-components)
```jsx
import { useState, useEffect } from 'react';
import TaskForm from './components/TaskForm';
import TaskItem from './components/TaskItem';

function App() {
  // Initialize state from LocalStorage if present
  const [tasks, setTasks] = useState(() => {
    const saved = localStorage.getItem('app_tasks');
    return saved ? JSON.parse(saved) : [
      { id: 1, text: 'Learn React Basics', completed: true },
      { id: 2, text: 'Build structured cheatsheet', completed: false }
    ];
  });

  const [filter, setFilter] = useState('all'); // all, active, completed

  // Persist tasks in localStorage whenever they change
  useEffect(() => {
    localStorage.setItem('app_tasks', JSON.stringify(tasks));
  }, [tasks]);

  const handleAddTask = (text) => {
    const newTask = {
      id: Date.now(),
      text,
      completed: false
    };
    setTasks((prev) => [...prev, newTask]);
  };

  const handleToggleComplete = (id) => {
    setTasks((prev) =>
      prev.map((task) =>
        task.id === id ? { ...task, completed: !task.completed } : task
      )
    );
  };

  const handleDeleteTask = (id) => {
    setTasks((prev) => prev.filter((task) => task.id !== id));
  };

  const handleClearCompleted = () => {
    setTasks((prev) => prev.filter((task) => !task.completed));
  };

  // Filter computation
  const filteredTasks = tasks.filter((task) => {
    if (filter === 'active') return !task.completed;
    if (filter === 'completed') return task.completed;
    return true; // 'all'
  });

  const activeCount = tasks.filter((task) => !task.completed).length;

  return (
    <div className="todo-app">
      <h1>Task Planner</h1>
      
      <TaskForm onAddTask={handleAddTask} />

      <div className="filter-controls">
        <button 
          onClick={() => setFilter('all')} 
          className={filter === 'all' ? 'active' : ''}
        >
          All
        </button>
        <button 
          onClick={() => setFilter('active')} 
          className={filter === 'active' ? 'active' : ''}
        >
          Active
        </button>
        <button 
          onClick={() => setFilter('completed')} 
          className={filter === 'completed' ? 'active' : ''}
        >
          Completed
        </button>
      </div>

      <div className="task-list">
        {filteredTasks.length === 0 ? (
          <p className="empty-message">No tasks in this section.</p>
        ) : (
          filteredTasks.map((task) => (
            <TaskItem
              key={task.id}
              task={task}
              onToggleComplete={handleToggleComplete}
              onDeleteTask={handleDeleteTask}
            />
          ))
        )}
      </div>

      <footer className="app-summary">
        <span>Items remaining: {activeCount}</span>
        {tasks.some(t => t.completed) && (
          <button onClick={handleClearCompleted} className="clear-btn">
            Clear Completed
          </button>
        )}
      </footer>
    </div>
  );
}

export default App;
```

---

### Key Takeaways and Rules
* **Functional State Setters**: Always update arrays cleanly using array spreads `[...prev, newItem]` or filtering `.filter()` to return new memory references. Do not use `.push()`, `.splice()`, or direct item mutation.
* **Lazy State Initialization**: Accessing `localStorage` can be expensive. Passing a function to `useState(() => { return value; })` ensures the initialization logic runs exactly once on mount, rather than on every re-render.
* **Derived State over Redundant State**: Do not store values like `filteredTasks` or `activeCount` in state variables. Instead, calculate them on-the-fly during render to keep state unified.

---

### Quick Reference Table
| Task Action | Array Operation Used | Returns |
| :--- | :--- | :--- |
| **Add Task** | `[...prev, newTask]` | A new array with the new element at the end |
| **Toggle Task** | `prev.map(t => t.id === id ? { ...t, completed: !t.completed } : t)` | A new array with the matched task object cloned and updated |
| **Delete Task** | `prev.filter(t => t.id !== id)` | A new array excluding the matched task |
| **Clear Completed** | `prev.filter(t => !t.completed)` | A new array excluding all completed task elements |

---

### Common Pitfalls and Anti-Patterns
#### 1. Mutating array elements directly inside state
* **Incorrect:**
  ```jsx
  const toggleComplete = (id) => {
    const matchedTask = tasks.find(t => t.id === id);
    matchedTask.completed = !matchedTask.completed; // Direct property mutation
    setTasks(tasks); // Passing the original array reference
  };
  ```
* **Why it fails:** Since the array reference `tasks` is unchanged, React skips re-rendering the list.
* **Correct:** Use `.map()` to build and return a completely new array with cloned modified objects:
  ```jsx
  const toggleComplete = (id) => {
    setTasks(prev => 
      prev.map(t => t.id === id ? { ...t, completed: !t.completed } : t)
    );
  };
  ```

#### 2. Creating separate state variables for derived counts
* **Incorrect:**
  ```jsx
  const [tasks, setTasks] = useState([]);
  const [activeCount, setActiveCount] = useState(0); // Redundant state
  ```
* **Why it fails:** You now have to manually update `activeCount` inside every single function that modifies `tasks`. If one updater fails to change the count, the UI displays outdated numbers.
* **Correct:** Simply derive the active task count directly during the render stage:
  ```jsx
  const activeCount = tasks.filter(t => !t.completed).length;
  ```
