# Module 6: Form Handling
---

### Core Concept
In React, forms are typically implemented using "controlled components." In a controlled component, the form's input elements are bound to the component state, making the React state the single source of truth. Any input change updates the state, and any state change reflects in the input value.

---

### Syntax and API Quick-Reference
Binding form state dynamically using a single handler:
```jsx
const [formData, setFormData] = useState({
  username: '',
  email: ''
});

const handleChange = (event) => {
  const { name, value } = event.target;
  setFormData((prevData) => ({
    ...prevData,
    [name]: value
  }));
};
```

---

### Production-Ready Example
Here is a comprehensive registration form component that handles multiple inputs (text, select, checkbox), manages submission, displays dynamic inline validation errors, and performs a clean state reset.

#### Component: `src/components/RegistrationForm.jsx`
```jsx
import { useState } from 'react';

function RegistrationForm() {
  const initialFormState = {
    username: '',
    email: '',
    role: 'developer',
    agreeToTerms: false
  };

  const [formData, setFormData] = useState(initialFormState);
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData((prev) => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const validateForm = () => {
    const tempErrors = {};
    if (!formData.username.trim()) {
      tempErrors.username = 'Username is required';
    }
    if (!formData.email.includes('@')) {
      tempErrors.email = 'Valid email is required';
    }
    if (!formData.agreeToTerms) {
      tempErrors.agreeToTerms = 'You must accept the terms';
    }
    setErrors(tempErrors);
    return Object.keys(tempErrors).length === 0;
  };

  const handleSubmit = (e) => {
    e.preventDefault(); // Stop page reload
    
    if (validateForm()) {
      console.log('Form Submitted successfully:', formData);
      alert('Registration Successful!');
      setFormData(initialFormState); // Reset form state
      setErrors({});
    }
  };

  return (
    <form onSubmit={handleSubmit} className="registration-form">
      <h2>User Registration</h2>

      <div className="form-group">
        <label htmlFor="username">Username:</label>
        <input
          type="text"
          id="username"
          name="username"
          value={formData.username}
          onChange={handleChange}
        />
        {errors.username && <span className="error-text">{errors.username}</span>}
      </div>

      <div className="form-group">
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
        {errors.email && <span className="error-text">{errors.email}</span>}
      </div>

      <div className="form-group">
        <label htmlFor="role">Role:</label>
        <select
          id="role"
          name="role"
          value={formData.role}
          onChange={handleChange}
        >
          <option value="developer">Developer</option>
          <option value="designer">Designer</option>
          <option value="manager">Project Manager</option>
        </select>
      </div>

      <div className="form-group checkbox-group">
        <input
          type="checkbox"
          id="agreeToTerms"
          name="agreeToTerms"
          checked={formData.agreeToTerms}
          onChange={handleChange}
        />
        <label htmlFor="agreeToTerms">I agree to the Terms of Service</label>
        {errors.agreeToTerms && <p className="error-text">{errors.agreeToTerms}</p>}
      </div>

      <button type="submit">Submit Registration</button>
    </form>
  );
}

export default RegistrationForm;
```

---

### Key Takeaways and Rules
* **e.preventDefault()**: Crucial to invoke inside the form's `onSubmit` handler to prevent the default browser behavior of reloading the page upon submission.
* **Controlled Inputs**: Always initialize form fields to non-null values (like an empty string `""` instead of `undefined`), or React will log a warning about switching from uncontrolled to controlled.
* **Dynamic Input Naming**: Giving inputs a `name` attribute matching the keys in the state object allows a single, elegant event handler function (`handleChange`) to handle any number of fields.

---

### Quick Reference Table
| Input Type | Prop to Bind State | Event Handler Event Type |
| :--- | :--- | :--- |
| Text, Email, Number | `value={stateVar}` | `onChange` -> `e.target.value` |
| Textarea | `value={stateVar}` | `onChange` -> `e.target.value` |
| Select (Dropdown) | `value={stateVar}` | `onChange` -> `e.target.value` |
| Checkbox, Radio | `checked={stateVar}` | `onChange` -> `e.target.checked` |

---

### Common Pitfalls and Anti-Patterns
#### 1. Losing other fields' data when updating a single form input
* **Incorrect:**
  ```jsx
  const handleChange = (e) => {
    // Missing spread operator (...prevData)
    setFormData({ [e.target.name]: e.target.value }); 
  };
  ```
* **Why it fails:** This completely overwrites the `formData` object, replacing all other input fields with just the single field that changed, causing those fields to disappear from state.
* **Correct:** Always merge the existing state using the spread operator before applying updates:
  ```jsx
  const handleChange = (e) => {
    setFormData(prev => ({
      ...prev,
      [e.target.name]: e.target.value
    }));
  };
  ```

#### 2. Initializing form values to `undefined` or `null`
* **Incorrect:**
  ```jsx
  const [formData, setFormData] = useState({
    username: undefined // Initialized as undefined
  });
  ```
* **Why it fails:** React views `value={undefined}` as uncontrolled. When the user types and state updates, the value becomes a string, triggering the warning: *“A component is changing an uncontrolled input to be controlled.”*
* **Correct:** Always initialize text fields to an empty string `""`:
  ```jsx
  const [formData, setFormData] = useState({
    username: ""
  });
  ```
