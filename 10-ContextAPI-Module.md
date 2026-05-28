# Module 10: Context API and Styling
---

### Core Concept
The React Context API solves the "props drilling" problem by providing a way to share state globally across the entire component tree without manually passing props down through every level. For styling, React apps can leverage scoping solutions like CSS Modules, utility systems like Tailwind CSS, or simple plain CSS.

---

### Syntax and API Quick-Reference
Creating and consuming Context:
```jsx
import { createContext, useContext, useState } from 'react';

// 1. Create the Context object
const AppContext = createContext();

// 2. Wrap tree with Provider in a parent component
export function AppProvider({ children }) {
  const [user, setUser] = useState('Tanmay');
  return (
    <AppContext.Provider value={{ user, setUser }}>
      {children}
    </AppContext.Provider>
  );
}

// 3. Consume the Context inside a child component
export function ChildComponent() {
  const { user } = useContext(AppContext);
  return <p>Active user: {user}</p>;
}
```

---

### Production-Ready Example
Here is a theme switching application combining React Context with CSS Modules and dynamic styles.

#### Context Setup: `src/context/ThemeContext.jsx`
```jsx
import { createContext, useState, useContext, useEffect } from 'react';

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState(() => {
    const saved = localStorage.getItem('app_theme');
    return saved || 'light';
  });

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  };

  useEffect(() => {
    localStorage.setItem('app_theme', theme);
    // Apply class to body for global styling
    document.body.className = theme === 'dark' ? 'dark-mode' : 'light-mode';
  }, [theme]);

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook to consume ThemeContext cleanly
export function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}
```

#### Consumption Component: `src/components/Navbar.jsx`
```jsx
import { useTheme } from '../context/ThemeContext';
import styles from './Navbar.module.css'; // Importing scoped CSS module

function Navbar() {
  const { theme, toggleTheme } = useTheme();

  return (
    <nav className={styles.navbar}>
      <span className={styles.logo}>AppLogo</span>
      <button onClick={toggleTheme} className={styles.themeBtn}>
        Switch to {theme === 'light' ? 'Dark' : 'Light'} Mode
      </button>
    </nav>
  );
}

export default Navbar;
```

#### Scoped Stylesheet: `src/components/Navbar.module.css`
```css
.navbar {
  display: flex;
  justify-content: space-between;
  padding: 1rem 2rem;
  background-color: var(--nav-bg, #f4f4f9);
  border-bottom: 1px solid #ddd;
}

.logo {
  font-weight: bold;
  font-size: 1.25rem;
}

.themeBtn {
  padding: 0.5rem 1rem;
  cursor: pointer;
  border-radius: 4px;
  border: 1px solid #ccc;
}
```

---

### Key Takeaways and Rules
* **Provider Wrapping**: The `<Context.Provider>` must wrap all child components that need access to the values. Placing it at the highest level (like `main.jsx`) makes its values global.
* **Custom Hook pattern**: Wrapping `useContext(MyContext)` inside a custom hook (e.g. `useTheme()`) is a best practice. It encapsulates context consumption and allows you to throw helpful errors if consumed outside a Provider.
* **CSS Scoping Options**:
  * **Plain CSS**: Easiest to write, but class names can clash globally across components.
  * **CSS Modules**: Automatically generates unique hash classes (e.g., `Navbar_logo__3x9d`), preventing styles from bleeding into other components.
  * **Tailwind CSS**: Offers utility-first class configurations without writing separate stylesheet files.

---

### Quick Reference Table
| Styling System | Installation Required | Key Advantage | Scoping Strategy |
| :--- | :--- | :--- | :--- |
| **Plain CSS** | No | Built-in out of the box | Global styles only |
| **CSS Modules** | No (built into Vite/CRA) | Zero naming clashes | Automatic unique class hashes |
| **Tailwind CSS** | Yes (`npm install -D tailwindcss`) | Extremely rapid UI assembly | Utility class compilation |

---

### Common Pitfalls and Anti-Patterns
#### 1. Consuming context without wrapping the component in a Provider
* **Incorrect:**
  ```jsx
  // App.jsx
  import { useTheme } from './context/ThemeContext';
  import ThemeToggle from './components/ThemeToggle';

  function App() {
    // Attempting to consume theme inside App itself
    const { theme } = useTheme(); 
    return <ThemeToggle />;
  }
  ```
* **Why it fails:** Since `App` is the component mounting the `ThemeProvider`, `App` itself sits outside the Provider context, causing the hook to return `undefined` and crash the application.
* **Correct:** Consume the context only inside child elements wrapped inside the Provider layout:
  ```jsx
  // App.jsx
  import { ThemeProvider } from './context/ThemeContext';
  import Navbar from './components/Navbar';

  function App() {
    return (
      <ThemeProvider>
        <Navbar /> {/* Consumes context safely inside */}
      </ThemeProvider>
    );
  }
  ```

#### 2. Re-rendering all consumers on unnecessary object reference updates
* **Incorrect:**
  ```jsx
  return (
    <UserContext.Provider value={{ user, settings }}>
      {children}
    </UserContext.Provider>
  );
  ```
* **Why it fails:** In this setup, every time the parent component renders for any reason, a brand new object `{ user, settings }` is created in memory. React sees this as a changed value reference, forcing all context consumers to re-render, even if `user` and `settings` didn't actually change.
* **Correct:** Use state or memoize the value object if rendering performance becomes critical:
  ```jsx
  const value = useMemo(() => ({ user, settings }), [user, settings]);
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
  ```
