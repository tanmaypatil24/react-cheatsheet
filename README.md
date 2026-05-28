# React Developer Cheatsheet

Welcome to the ultimate, structured React Developer Cheatsheet. This repository contains a comprehensive, study-optimized guide designed for quick reference, rapid interview prep, and production-grade code pattern lookup.

All material is divided into a progressive 15-module syllabus covering both foundational concepts and advanced performance and testing strategies.

---

## Quick Navigation Index

### Beginner Track
* [Module 1: Setup and Basics](file:///f:/Github/react-cheatsheet/1-React-Setup.md) - Node environment, Vite vs Create React App, folder structure, and clearing boilerplate.
* [Module 2: JSX and Components](file:///f:/Github/react-cheatsheet/2-JSX-Components.md) - Syntax rules, functional components, dynamic expressions, props, and default values.
* [Module 3: State and Events](file:///f:/Github/react-cheatsheet/3-State-And-Events.md) - State concept, useState hook, event listener binding, input changes, and stale state prevention.
* [Module 4: Conditional Rendering and Lists](file:///f:/Github/react-cheatsheet/4-Conditional-Rendering-And-Lists.md) - Ternary conditions, double-ampersand short-circuits, array mapping, and unique key props.
* [Module 5: useEffect and Lifecycle](file:///f:/Github/react-cheatsheet/5-useEffect-And-Lifecycle.md) - Side-effect synchronization, empty/active dependency arrays, data fetching, and abort controllers.
* [Module 6: Form Handling](file:///f:/Github/react-cheatsheet/6-Form-Handling.md) - Controlled components, multi-input dynamic handlers, validation, and state reset triggers.
* [Module 7: Routing with React Router](file:///f:/Github/react-cheatsheet/7-Routing-With-React-Router.md) - SPA navigation, Router setup, NavLink visual states, URL parameters, and programmatic navigate.
* [Module 8: Component Communication](file:///f:/Github/react-cheatsheet/8-Component-Communication.md) - Parent-to-child data flow, lifting child state up via callbacks, sibling sync, and props drilling.
* [Module 9: Basic Mini Project](file:///f:/Github/react-cheatsheet/9-Basic-Project.md) - Practical task manager app featuring local storage synchronization and interactive filters.
* [Module 10: Context API and Styling](file:///f:/Github/react-cheatsheet/10-ContextAPI-Module.md) - Context state providers, useContext, CSS Modules, and Tailwind CSS.

### Advanced Track
* [Module 11: useReducer and Complex State](file:///f:/Github/react-cheatsheet/11-useReducer-And-Complex-State-Management.md) - Dispatchers, actions, pure reducer functions, and state comparison guides.
* [Module 12: React Performance Optimization](file:///f:/Github/react-cheatsheet/12-React-Perfomance-Optimization.md) - React.memo rendering skips, CPU computation caching with useMemo, and reference stability with useCallback.
* [Module 13: Custom Hooks](file:///f:/Github/react-cheatsheet/13-React-Custom-Hooks.md) - Extracting stateful logic, hooks rules, and custom recipes (useFetch, useLocalStorage).
* [Module 14: React Lazy Loading and Suspense](file:///f:/Github/react-cheatsheet/14-React-Lazy-Loading.md) - Route-level code splitting, dynamic imports, bundle size optimizations, and fallback UI wrappers.
* [Module 15: React Testing](file:///f:/Github/react-cheatsheet/15-React-Testing-With-Jest.md) - Jest test suites, RTL virtual mounting, query systems, interactive user event simulations, and API fetch mocking.

---

## Core Hooks Quick-Reference Table

Use this summary table for quick API syntax lookup:

| Hook | Purpose | Common Syntax | Dependency Array |
| :--- | :--- | :--- | :--- |
| **useState** | Dynamic local component state | `const [val, setVal] = useState(init);` | No |
| **useEffect** | External side-effects and lifecycle | `useEffect(() => { return () => cleanup(); }, [deps]);` | Yes (optional) |
| **useContext** | Global state consumer | `const contextVal = useContext(MyContext);` | No |
| **useReducer** | Complex state transaction management | `const [state, dispatch] = useReducer(reducer, init);` | No |
| **useMemo** | Caching calculated values | `const val = useMemo(() => calculate(a, b), [a, b]);` | Yes |
| **useCallback** | Maintaining function referential stability | `const fn = useCallback(() => handler(a), [a]);` | Yes |
| **useParams** | Query dynamic URL route parameters | `const { userId } = useParams();` | No |
| **useNavigate** | Programmatic SPA page redirection | `const navigate = useNavigate();` | No |

---

## Getting Started Quickly

### 1. Scaffold the Project
Initialize a modern, high-speed project utilizing the Vite bundler:
```bash
npm create vite@latest my-react-app -- --template react
cd my-react-app
```

### 2. Install Standard Dependencies
Optionally install routing, utility styling, and testing libraries if proceeding with advanced modules:
```bash
# Routing
npm install react-router-dom

# Tailwind CSS (Utility Styling)
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Testing
npm install -D @testing-library/react @testing-library/user-event @testing-library/jest-dom jest
```

### 3. Run Local Development Server
Boot up the Vite build-and-run system:
```bash
npm install
npm run dev
```
Open your browser to the displayed URL (typically `http://localhost:5173`) to view your running application.