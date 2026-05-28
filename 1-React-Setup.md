# Module 1: Setup and Basics
---

### Core Concept
React is a component-based JavaScript library for building user interfaces. To run React locally, Node.js and npm are required to manage dependencies and run the local development server. Modern React projects typically use Vite for lightning-fast build speeds, while legacy projects may use Create React App (CRA).

---

### Syntax and API Quick-Reference
Verify installation of Node.js and npm:
```bash
node -v
npm -v
```

Create a new project using Vite (Recommended):
```bash
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
npm run dev
```

Create a new project using Create React App (Legacy):
```bash
npx create-react-app my-react-app
cd my-react-app
npm start
```

---

### Production-Ready Example
Clean modern React entry-point boilerplate:

#### Entry Point: `src/main.jsx` (Vite) or `src/index.js` (CRA)
```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

const rootElement = document.getElementById('root');
const root = ReactDOM.createRoot(rootElement);

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

#### Main Component: `src/App.jsx`
```jsx
function App() {
  return (
    <div>
      <h1>Hello React</h1>
    </div>
  );
}

export default App;
```

---

### Key Takeaways and Rules
* **Node.js Environment**: Node.js is required for tooling, but React itself runs entirely in the client browser.
* **Vite vs CRA**: Vite is the modern standard, offering near-instant hot module replacement (HMR) and smaller build sizes compared to Create React App.
* **StrictMode**: `<React.StrictMode>` is a development-only helper that runs side-effects twice to help identify memory leaks and unsafe lifecycles.
* **Folder Structure**:
  * `public/`: Contains static assets like HTML template (`index.html`) and icons.
  * `src/`: Contains all React components, styles, and assets that will be compiled.

---

### Quick Reference Table
| Tool / Command | Command Syntax | Description |
| :--- | :--- | :--- |
| Node.js / npm | `node -v` / `npm -v` | Checks installed Node.js and package manager versions |
| Vite Creation | `npm create vite@latest [name]` | Scaffolds a new Vite project quickly |
| Install Packages | `npm install` | Installs project dependencies defined in package.json |
| Vite Dev Server | `npm run dev` | Starts Vite local development server (typically localhost:5173) |
| CRA Dev Server | `npm start` | Starts CRA local development server (typically localhost:3000) |

---

### Common Pitfalls and Anti-Patterns
#### 1. Running React tooling without Node.js installed
* **Incorrect:** Trying to run `npx` or `npm` commands directly in terminal without having Node.js configured in system environment variables.
* **Why it fails:** Systems cannot resolve `npm` or `npx` unless Node.js is installed and added to the PATH.
* **Correct:** Download the LTS installer from nodejs.org, complete setup, and verify with `node -v` and `npm -v`.

#### 2. Storing active React source code in the `public` directory
* **Incorrect:** Placing custom JavaScript files or CSS that need compilation into `public/`.
* **Why it fails:** Files in `public` are copied directly to the build output without any optimization, bundling, or transpiling.
* **Correct:** Place all components, CSS, and dynamic assets in the `src/` directory.