# Module 15: React Testing
---

### Core Concept
Testing ensures your components function correctly when subjected to different props and user interactions. We use Jest as our test runner and assertion library, and React Testing Library (RTL) to render components into a virtual DOM. RTL encourages testing components as if you were a real user, rather than testing internal implementation details.

---

### Syntax and API Quick-Reference
Standard Test File Structure (`.test.js` or `.spec.js`):
```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import MyButton from './MyButton';

describe('MyButton Component tests', () => {
  test('should render button with correct text label', () => {
    // 1. Render component inside virtual DOM
    render(<MyButton label="Click Me" />);

    // 2. Query element in the virtual DOM
    const btnElement = screen.getByText('Click Me');

    // 3. Make assertion on the element
    expect(btnElement).toBeInTheDocument();
  });
});
```

---

### Production-Ready Example
Here is a complete test suite for an interactive counter component that asserts initial render states, clicks, and state changes.

#### Component to Test: `src/components/Counter.jsx`
```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div className="counter-box">
      <h2>Points: <span data-testid="count-val">{count}</span></h2>
      <button onClick={() => setCount(prev => prev + 1)}>Increment</button>
      <button onClick={() => setCount(prev => prev - 1)}>Decrement</button>
    </div>
  );
}

export default Counter;
```

#### Test Suite: `src/components/Counter.test.jsx`
```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Counter from './Counter';

describe('Counter Interactive Tests', () => {
  test('should render initial score value of 0', () => {
    render(<Counter />);
    const scoreVal = screen.getByTestId('count-val');
    expect(scoreVal).toHaveTextContent('0');
  });

  test('should increase count by 1 when increment button is clicked', async () => {
    render(<Counter />);
    
    // Set up user event session
    const user = userEvent.setup();
    
    const incrementBtn = screen.getByRole('button', { name: /increment/i });
    const scoreVal = screen.getByTestId('count-val');

    // Perform interactive click
    await user.click(incrementBtn);

    expect(scoreVal).toHaveTextContent('1');
  });

  test('should decrease count by 1 when decrement button is clicked', async () => {
    render(<Counter />);
    
    const user = userEvent.setup();
    const decrementBtn = screen.getByRole('button', { name: /decrement/i });
    const scoreVal = screen.getByTestId('count-val');

    await user.click(decrementBtn);

    expect(scoreVal).toHaveTextContent('-1');
  });
});
```

---

### Key Takeaways and Rules
* **Accessibility-First Queries**: Always query elements using user-facing selectors. Prefer `.getByRole()` or `.getByText()` over custom `.getByTestId()` or CSS classes. This ensures your components remain accessible to screen readers.
* **userEvent vs fireEvent**: Use `@testing-library/user-event` instead of `fireEvent` where possible. `userEvent` simulates complete, realistic browser interactions (such as hover events, keyups, and focus triggers accompanying a click).
* **Mocking API calls**: Never make real network requests in your test suite. Mock global `fetch` using Jest spies or set up Mock Service Worker (MSW) to intercept and respond to mock API requests.

---

### Quick Reference Table
| Query Prefix | Returns | Behavior if Element Not Found |
| :--- | :--- | :--- |
| `getBy...` | Matched HTML Element | Throws immediate fatal error (Ideal for absolute assertions) |
| `queryBy...` | Matched HTML Element or `null` | Returns `null` (Ideal for asserting elements are *not* present) |
| `findBy...` | Promise resolving element | Retries query for 1000ms before failing (Ideal for async elements) |

---

### Common Pitfalls and Anti-Patterns
#### 1. Asserting non-existence using `getBy...` methods
* **Incorrect:**
  ```jsx
  // Expecting loading screen to have disappeared
  const spinner = screen.getByText(/loading/i); 
  expect(spinner).not.toBeInTheDocument();
  ```
* **Why it fails:** Since the element is not found, `getByText` throws a fatal error immediately, crashing the test before `expect()` is even evaluated.
* **Correct:** Use `queryBy...` methods which return `null` instead of throwing an error:
  ```jsx
  const spinner = screen.queryByText(/loading/i);
  expect(spinner).not.toBeInTheDocument(); // Evaluates cleanly to true
  ```

#### 2. Forgetting to wait for asynchronous changes when testing API fetches
* **Incorrect:**
  ```jsx
  render(<UserList />);
  const userItem = screen.getByText('Tanmay'); // Fetching is async; element not present immediately
  expect(userItem).toBeInTheDocument();
  ```
* **Why it fails:** React performs data fetching asynchronously. The test runner checks the DOM immediately after rendering before the mock fetch call completes, and throws an error.
* **Correct:** Use a `findBy...` query which returns a promise and waits for the UI to update:
  ```jsx
  render(<UserList />);
  const userItem = await screen.findByText('Tanmay'); // Waits for render update
  expect(userItem).toBeInTheDocument();
  ```
