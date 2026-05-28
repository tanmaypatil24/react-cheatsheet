# Module 8: Component Communication
---

### Core Concept
In React, data flows unidirectionally from parent components to child components via props ("top-down"). When a child component needs to communicate updates back to its parent, it does so by invoking callback functions passed down as props from the parent. This mechanism is known as "lifting state up."

---

### Syntax and API Quick-Reference
Lifting State Up (Child-to-Parent Communication):

#### Parent Component Setup:
```jsx
const [data, setData] = useState('');

const handleUpdate = (newValue) => {
  setData(newValue);
};

return <Child onUpdate={handleUpdate} />;
```

#### Child Component Setup:
```jsx
function Child({ onUpdate }) {
  return (
    <button onClick={() => onUpdate('Data from Child')}>
      Send Data
    </button>
  );
}
```

---

### Production-Ready Example
Let's create a coordinate system where a sibling control panel updates the coordinate state inside a parent component, which is then rendered by another sibling display component.

#### Parent Component: `src/components/ParentController.jsx`
```jsx
import { useState } from 'react';
import SiblingDisplay from './SiblingDisplay';
import SiblingControls from './SiblingControls';

function ParentController() {
  const [coordinates, setCoordinates] = useState({ x: 0, y: 0 });

  const handleMoveUp = () => {
    setCoordinates((prev) => ({ ...prev, y: prev.y + 1 }));
  };

  const handleMoveDown = () => {
    setCoordinates((prev) => ({ ...prev, y: prev.y - 1 }));
  };

  const handleReset = () => {
    setCoordinates({ x: 0, y: 0 });
  };

  return (
    <div className="controller-board">
      <h2>Game Coordinate System</h2>
      
      {/* Sibling 1: Displays state passed down from parent */}
      <SiblingDisplay coords={coordinates} />

      {/* Sibling 2: Updates parent state via callback props */}
      <SiblingControls 
        onUp={handleMoveUp} 
        onDown={handleMoveDown} 
        onReset={handleReset} 
      />
    </div>
  );
}

export default ParentController;
```

#### Sibling 1: `src/components/SiblingDisplay.jsx`
```jsx
function SiblingDisplay({ coords }) {
  return (
    <div className="display-panel">
      <h3>Position Dashboard</h3>
      <p>X Coordinate: <strong>{coords.x}</strong></p>
      <p>Y Coordinate: <strong>{coords.y}</strong></p>
    </div>
  );
}

export default SiblingDisplay;
```

#### Sibling 2: `src/components/SiblingControls.jsx`
```jsx
function SiblingControls({ onUp, onDown, onReset }) {
  return (
    <div className="controls-panel">
      <h3>Control Console</h3>
      <button onClick={onUp}>Move Up (Y+1)</button>
      <button onClick={onDown}>Move Down (Y-1)</button>
      <button onClick={onReset}>Return to Center</button>
    </div>
  );
}

export default SiblingControls;
```

---

### Key Takeaways and Rules
* **Unidirectional Flow**: State always belongs to a single owner. Components should only read props and trigger functions; they should never mutate incoming values directly.
* **Lifting State Up**: When two or more sibling components need access to the same state data, define that state in their nearest common ancestor parent and pass it down as props.
* **Callbacks for Upward Flow**: Sibling-to-sibling communication does not exist directly. Sibling A must update Parent state via a callback, and Parent then flows that updated state down to Sibling B.

---

### Quick Reference Table
| Direction of Flow | Mechanism Used | Description |
| :--- | :--- | :--- |
| **Parent -> Child** | Props | Normal top-down attribute passing |
| **Child -> Parent** | Callback Functions | Parent passes a function; Child executes it with arguments |
| **Sibling -> Sibling** | Lifting State to Ancestor | Coordinate state inside parent; communicate through parent |
| **Deep Component Nesting** | Context API / Redux | Solves "Props Drilling" through multiple intermediate layers |

---

### Common Pitfalls and Anti-Patterns
#### 1. "Props Drilling" through too many structural layout layers
* **Incorrect:**
  ```jsx
  // Passing user authentication state down 6 layers of structural header wrappers
  <App> -> <Layout> -> <Header> -> <Navbar> -> <UserMenu> -> <UserAvatar user={user} />
  ```
* **Why it fails:** Intermediate components (`Layout`, `Header`, `Navbar`, etc.) do not need or use the `user` prop. They are forced to pass it along anyway, creating highly coupled, brittle, and unmaintainable code.
* **Correct:** For highly global state, use the Context API (Module 10) to let `UserAvatar` consume the authenticated state directly.

#### 2. Replicating props into local state in the child component unnecessarily
* **Incorrect:**
  ```jsx
  function Child({ value }) {
    const [localVal, setLocalVal] = useState(value); // Mirroring prop in state
    return <div>{localVal}</div>;
  }
  ```
* **Why it fails:** If the parent updates `value` later, the child will continue displaying the old initial `localVal` since the local state constructor only runs once when mounting.
* **Correct:** Use the prop directly inside the rendering tree instead of duplicating it in state:
  ```jsx
  function Child({ value }) {
    return <div>{value}</div>;
  }
  ```
