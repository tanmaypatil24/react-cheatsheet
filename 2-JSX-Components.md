# Module 2: JSX and Components
---

### Core Concept
JSX (JavaScript XML) is a syntax extension for JavaScript that allows you to write HTML-like structures inside JavaScript files. React components are reusable JavaScript functions that return JSX. Components can accept inputs called "props" (properties) to customize their rendering.

---

### Syntax and API Quick-Reference
Basic Functional Component with Props destructuring:
```jsx
function WelcomeCard({ name, role = "User" }) {
  return (
    <div className="card">
      <h2>Welcome, {name}</h2>
      <p>Role: {role}</p>
    </div>
  );
}
```

---

### Production-Ready Example
Let's build a modular layout with multiple components communicating via props.

#### Component: `src/components/Header.jsx`
```jsx
function Header({ title, subtitle }) {
  return (
    <header className="site-header">
      <h1>{title}</h1>
      {subtitle && <p>{subtitle}</p>}
    </header>
  );
}

export default Header;
```

#### Component: `src/components/UserCard.jsx`
```jsx
function UserCard({ name, email, isActive }) {
  return (
    <div className={`user-card ${isActive ? 'active' : 'inactive'}`}>
      <h3>{name}</h3>
      <p>Email: {email}</p>
      <span className="status-badge">
        {isActive ? "Active User" : "Inactive"}
      </span>
    </div>
  );
}

export default UserCard;
```

#### Root Component: `src/App.jsx`
```jsx
import Header from './components/Header';
import UserCard from './components/UserCard';

function App() {
  return (
    <div className="app-container">
      <Header 
        title="Admin Dashboard" 
        subtitle="Manage registered system users" 
      />
      <main className="content">
        <UserCard name="Tanmay Patil" email="tanmay@example.com" isActive={true} />
        <UserCard name="Amit Sharma" email="amit@example.com" isActive={false} />
      </main>
    </div>
  );
}

export default App;
```

---

### Key Takeaways and Rules
* **Single Root Element**: JSX must return a single root element (or a Fragment `<></>`).
* **Closing Tags**: All tags must be explicitly closed. Self-closing tags require a trailing slash (e.g., `<img />`, `<br />`, `<input />`).
* **camelCase Attributes**: HTML attribute names become camelCase in JSX (e.g., `class` becomes `className`, `onclick` becomes `onClick`, `for` becomes `htmlFor`).
* **Expression Evaluation**: Any valid JavaScript expression can be embedded inside JSX using curly braces `{}`.
* **Props are Read-Only (Immutable)**: A component must never modify its own props. Props should only be updated by the parent passing them down.

---

### Quick Reference Table
| Concept | Syntax | Description |
| :--- | :--- | :--- |
| React Fragment | `<> ... </>` or `<React.Fragment> ... </React.Fragment>` | Groups multiple elements without adding an extra node to the DOM |
| Props Destructuring | `function MyComponent({ propA, propB })` | Directly extracts properties from the props object in the function signature |
| Default Props | `function MyComponent({ role = "User" })` | Sets a fallback value if a prop is not provided by the parent |
| Dynamic Classes | `className={isActive ? "active" : "inactive"}` | Uses ternary operator to apply CSS classes conditionally |

---

### Common Pitfalls and Anti-Patterns
#### 1. Modifying props inside a child component
* **Incorrect:**
  ```jsx
  function UserProfile(props) {
    props.username = "NewName"; // Mutating props directly
    return <div>{props.username}</div>;
  }
  ```
* **Why it fails:** React enforces strict immutability for props. Direct mutations break React's rendering lifecycle and state tracking.
* **Correct:** Use state in the parent component and pass an event handler function to change it.
  ```jsx
  function UserProfile({ username, onUpdateName }) {
    return (
      <div>
        <p>{username}</p>
        <button onClick={() => onUpdateName("NewName")}>Change Name</button>
      </div>
    );
  }
  ```

#### 2. Returning multiple root elements in JSX
* **Incorrect:**
  ```jsx
  function NavMenu() {
    return (
      <a href="#home">Home</a>
      <a href="#about">About</a>
    );
  }
  ```
* **Why it fails:** JSX compiles to standard JavaScript function calls (`React.createElement`). A function cannot return two values simultaneously without wrapping them in an array or object.
* **Correct:** Wrap adjacent elements in a React Fragment:
  ```jsx
  function NavMenu() {
    return (
      <>
        <a href="#home">Home</a>
        <a href="#about">About</a>
      </>
    );
  }
  ```
