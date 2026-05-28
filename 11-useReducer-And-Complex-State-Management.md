# Module 11: useReducer and Complex State Management
---

### Core Concept
The `useReducer` hook is an alternative to `useState` that is better suited for managing complex state objects, deeply nested states, or states where the next value depends heavily on the previous value. It uses the action-reducer pattern popularized by Redux, keeping state transition logic organized and separated from rendering logic.

---

### Syntax and API Quick-Reference
Initializing `useReducer`:
```jsx
import { useReducer } from 'react';

// 1. Define initial state
const initialState = { count: 0 };

// 2. Define reducer function containing logic transitions
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error(`Unhandled action type: ${action.type}`);
  }
}

// 3. Destructure state and dispatch in the component
const [state, dispatch] = useReducer(reducer, initialState);
```

---

### Production-Ready Example
Here is a form management setup with input tracking, errors, and validation states using `useReducer`.

#### Component: `src/components/FormManager.jsx`
```jsx
import { useReducer } from 'react';

const initialState = {
  inputs: { username: '', email: '' },
  errors: { username: '', email: '' },
  isSubmitting: false
};

function formReducer(state, action) {
  switch (action.type) {
    case 'UPDATE_INPUT':
      return {
        ...state,
        inputs: {
          ...state.inputs,
          [action.field]: action.value
        },
        // Clear error when user modifies field
        errors: {
          ...state.errors,
          [action.field]: ''
        }
      };
    case 'SET_ERRORS':
      return {
        ...state,
        errors: action.payload,
        isSubmitting: false
      };
    case 'START_SUBMIT':
      return {
        ...state,
        isSubmitting: true
      };
    case 'RESET_FORM':
      return initialState;
    default:
      return state;
  }
}

function FormManager() {
  const [state, dispatch] = useReducer(formReducer, initialState);

  const handleInputChange = (e) => {
    dispatch({
      type: 'UPDATE_INPUT',
      field: e.target.name,
      value: e.target.value
    });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    dispatch({ type: 'START_SUBMIT' });

    const tempErrors = {};
    if (!state.inputs.username.trim()) {
      tempErrors.username = 'Username cannot be blank';
    }
    if (!state.inputs.email.includes('@')) {
      tempErrors.email = 'Email must be valid';
    }

    if (Object.keys(tempErrors).length > 0) {
      dispatch({ type: 'SET_ERRORS', payload: tempErrors });
    } else {
      console.log('Sending data:', state.inputs);
      alert('Form sent successfully!');
      dispatch({ type: 'RESET_FORM' });
    }
  };

  return (
    <form onSubmit={handleSubmit} className="reducer-form">
      <h2>Advanced Form Registration</h2>

      <div className="form-group">
        <label>Username:</label>
        <input
          type="text"
          name="username"
          value={state.inputs.username}
          onChange={handleInputChange}
        />
        {state.errors.username && <span className="error">{state.errors.username}</span>}
      </div>

      <div className="form-group">
        <label>Email Address:</label>
        <input
          type="email"
          name="email"
          value={state.inputs.email}
          onChange={handleInputChange}
        />
        {state.errors.email && <span className="error">{state.errors.email}</span>}
      </div>

      <button type="submit" disabled={state.isSubmitting}>
        {state.isSubmitting ? 'Processing...' : 'Register'}
      </button>
    </form>
  );
}

export default FormManager;
```

---

### Key Takeaways and Rules
* **Pure Reducer Functions**: The `reducer` function must be a "pure function." It must take in `(state, action)` and return a completely *new* state object. It must never perform side-effects (like API calls) or mutate the existing state directly.
* **Separation of Concerns**: Moving complex state transition rules out of components makes testing easier and leaves components focused purely on rendering the user interface.
* **Action Types Standard**: Actions should be structured objects, typically possessing a `type` string (e.g. `'UPDATE_INPUT'`) and an optional data load property commonly named `payload` or specific parameters.

---

### Quick Reference Table
| Selection Guide | `useState` | `useReducer` |
| :--- | :--- | :--- |
| **State Type** | Primitive types, simple objects, or arrays | Complex nested objects, multiple interrelated state variables |
| **Logic Location** | Inline inside event handler functions | Centralized inside a standalone pure reducer function |
| **Transition Complexity** | Low (simple reassignment) | High (many different business rules or triggers) |
| **Testing** | Coupled with component rendering | Simple to test independently (just passing input/action to function) |

---

### Common Pitfalls and Anti-Patterns
#### 1. Mutating state objects directly inside a reducer switch statement
* **Incorrect:**
  ```jsx
  function reducer(state, action) {
    switch (action.type) {
      case 'increment':
        state.count = state.count + 1; // Mutating original state object
        return state;
    }
  }
  ```
* **Why it fails:** React performs shallow equality checks on state references. Since the returned state is the same reference object as the input state, React assumes nothing has changed and skips updating the DOM.
* **Correct:** Always return a brand new object created using structural spread copies:
  ```jsx
  function reducer(state, action) {
    switch (action.type) {
      case 'increment':
        return {
          ...state,
          count: state.count + 1 // New reference returned
        };
    }
  }
  ```

#### 2. Placing network requests or side-effects inside the reducer
* **Incorrect:**
  ```jsx
  function reducer(state, action) {
    switch (action.type) {
      case 'fetch_user':
        fetch('/user').then(res => dispatch({ type: 'success', data: res })); // Side effect
        return { ...state, loading: true };
    }
  }
  ```
* **Why it fails:** Reducers must be entirely deterministic and free from external side-effects. Side-effects break time-travel debugging and can lead to duplicate network calls and unpredictable UI behavior.
* **Correct:** Perform all API calls inside your component event handlers or `useEffect` hooks, then `dispatch` clean structured actions containing the results:
  ```jsx
  const handleFetch = async () => {
    dispatch({ type: 'START_FETCH' });
    try {
      const data = await apiCall();
      dispatch({ type: 'FETCH_SUCCESS', payload: data });
    } catch (err) {
      dispatch({ type: 'FETCH_ERROR', payload: err.message });
    }
  };
  ```
