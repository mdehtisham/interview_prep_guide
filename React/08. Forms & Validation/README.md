# Forms & Validation

> Intermediate / Mid-Level (1-3 years)

---

## Questions

71. What is React Hook Form and why use it?
72. What is Formik library?
73. How do you implement form validation in React?
74. What is Yup validation library?
75. How do you handle file uploads in React?
76. What is Zod and how do you use it with forms?
77. How do you implement multi-step forms?
78. What are controlled vs uncontrolled forms?
79. How do you handle form submission in React?
80. What is the difference between onChange and onBlur?

---

## Detailed Answers

### 71. What is React Hook Form and why use it?

<details>
<summary>View Answer</summary>

**React Hook Form**

React Hook Form is a **performant, flexible, and extensible** form library for React with **easy-to-use validation**. It uses **uncontrolled components** and **refs** to minimize re-renders.

**Official website:** https://react-hook-form.com/

---

## Why Use React Hook Form?

**Problems with traditional form handling:**
```jsx
// Traditional approach - lots of state and re-renders
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  
  // Re-renders on every keystroke!
  return (
    <form>
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
    </form>
  );
}
```

**With React Hook Form:**
```jsx
import { useForm } from 'react-hook-form';

function LoginForm() {
  const { register, handleSubmit } = useForm();
  
  const onSubmit = (data) => console.log(data);
  
  // Minimal re-renders!
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      <input {...register('password')} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Key Benefits

**1. Performance**
- Minimal re-renders
- Uncontrolled components (uses refs)
- Faster than Formik
- ~9KB minified + gzipped

**2. Developer Experience**
- Simple API
- Less code
- TypeScript support
- DevTools available

**3. Validation**
- Built-in validation
- Schema validation (Yup, Zod, Joi)
- Async validation
- Field-level errors

**4. Flexibility**
- Works with UI libraries
- Custom components
- Controlled/uncontrolled mix
- No dependencies

---

## Installation

```bash
npm install react-hook-form
```

---

## Basic Usage

### Simple Form

```jsx
import { useForm } from 'react-hook-form';

function ContactForm() {
  const { register, handleSubmit } = useForm();
  
  const onSubmit = (data) => {
    console.log(data);
    // { name: 'John', email: 'john@example.com', message: 'Hello' }
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} placeholder="Name" />
      <input {...register('email')} placeholder="Email" />
      <textarea {...register('message')} placeholder="Message" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

### Form with Validation

```jsx
import { useForm } from 'react-hook-form';

function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors }
  } = useForm();
  
  const onSubmit = (data) => {
    console.log(data);
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Email with validation */}
      <input
        {...register('email', {
          required: 'Email is required',
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: 'Invalid email address'
          }
        })}
        placeholder="Email"
      />
      {errors.email && <p>{errors.email.message}</p>}
      
      {/* Password with validation */}
      <input
        type="password"
        {...register('password', {
          required: 'Password is required',
          minLength: {
            value: 8,
            message: 'Password must be at least 8 characters'
          }
        })}
        placeholder="Password"
      />
      {errors.password && <p>{errors.password.message}</p>}
      
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

---

## Core Concepts

### 1. register()

**Registers input and applies validation:**

```jsx
const { register } = useForm();

// Basic registration
<input {...register('firstName')} />

// With validation
<input
  {...register('age', {
    required: true,
    min: 18,
    max: 99
  })}
/>
```

**Register options:**
```jsx
register('fieldName', {
  required: 'This field is required',
  minLength: { value: 5, message: 'Min 5 characters' },
  maxLength: { value: 20, message: 'Max 20 characters' },
  min: { value: 18, message: 'Must be 18+' },
  max: { value: 99, message: 'Must be under 100' },
  pattern: { value: /regex/, message: 'Invalid format' },
  validate: (value) => value !== 'admin' || 'Username taken'
})
```

---

### 2. handleSubmit()

**Handles form submission with validation:**

```jsx
const { handleSubmit } = useForm();

const onSubmit = (data) => {
  console.log(data);  // Only called if valid
};

const onError = (errors) => {
  console.log(errors);  // Called if validation fails
};

<form onSubmit={handleSubmit(onSubmit, onError)}>
  {/* form fields */}
</form>
```

---

### 3. formState

**Access form state:**

```jsx
const {
  formState: {
    errors,        // Validation errors
    isDirty,       // Form has been modified
    isValid,       // Form is valid
    isSubmitting,  // Form is being submitted
    touchedFields, // Fields that have been touched
    dirtyFields,   // Fields that have been modified
    submitCount    // Number of times submitted
  }
} = useForm();
```

---

### 4. watch()

**Watch field values:**

```jsx
const { register, watch } = useForm();

const email = watch('email');  // Watch single field
const allValues = watch();     // Watch all fields

return (
  <div>
    <input {...register('email')} />
    <p>You typed: {email}</p>
  </div>
);
```

---

### 5. setValue()

**Set field value programmatically:**

```jsx
const { register, setValue } = useForm();

const fillForm = () => {
  setValue('name', 'John Doe');
  setValue('email', 'john@example.com');
};

return (
  <div>
    <button onClick={fillForm}>Fill Form</button>
    <input {...register('name')} />
    <input {...register('email')} />
  </div>
);
```

---

## Complete Example: User Registration

```jsx
import React, { useState } from 'react';
import { useForm } from 'react-hook-form';
import './SignupForm.css';

function SignupForm() {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors, isSubmitting, isDirty, isValid }
  } = useForm({
    mode: 'onChange'  // Validate on change
  });
  
  const [submitSuccess, setSubmitSuccess] = useState(false);
  
  const password = watch('password');  // For confirm password validation
  
  const onSubmit = async (data) => {
    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      console.log('Form Data:', data);
      setSubmitSuccess(true);
    } catch (error) {
      console.error('Submission error:', error);
    }
  };
  
  if (submitSuccess) {
    return (
      <div className="success-message">
        <h2>Registration Successful!</h2>
        <p>Welcome, {watch('username')}!</p>
      </div>
    );
  }
  
  return (
    <div className="signup-form">
      <h1>Sign Up</h1>
      
      <form onSubmit={handleSubmit(onSubmit)}>
        {/* Username */}
        <div className="form-group">
          <label htmlFor="username">Username</label>
          <input
            id="username"
            {...register('username', {
              required: 'Username is required',
              minLength: {
                value: 3,
                message: 'Username must be at least 3 characters'
              },
              maxLength: {
                value: 20,
                message: 'Username must be less than 20 characters'
              },
              pattern: {
                value: /^[a-zA-Z0-9_]+$/,
                message: 'Username can only contain letters, numbers, and underscores'
              }
            })}
            placeholder="Enter username"
            className={errors.username ? 'error' : ''}
          />
          {errors.username && (
            <span className="error-message">{errors.username.message}</span>
          )}
        </div>
        
        {/* Email */}
        <div className="form-group">
          <label htmlFor="email">Email</label>
          <input
            id="email"
            type="email"
            {...register('email', {
              required: 'Email is required',
              pattern: {
                value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
                message: 'Invalid email address'
              }
            })}
            placeholder="Enter email"
            className={errors.email ? 'error' : ''}
          />
          {errors.email && (
            <span className="error-message">{errors.email.message}</span>
          )}
        </div>
        
        {/* Password */}
        <div className="form-group">
          <label htmlFor="password">Password</label>
          <input
            id="password"
            type="password"
            {...register('password', {
              required: 'Password is required',
              minLength: {
                value: 8,
                message: 'Password must be at least 8 characters'
              },
              pattern: {
                value: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
                message: 'Password must contain uppercase, lowercase, and number'
              }
            })}
            placeholder="Enter password"
            className={errors.password ? 'error' : ''}
          />
          {errors.password && (
            <span className="error-message">{errors.password.message}</span>
          )}
        </div>
        
        {/* Confirm Password */}
        <div className="form-group">
          <label htmlFor="confirmPassword">Confirm Password</label>
          <input
            id="confirmPassword"
            type="password"
            {...register('confirmPassword', {
              required: 'Please confirm your password',
              validate: value =>
                value === password || 'Passwords do not match'
            })}
            placeholder="Confirm password"
            className={errors.confirmPassword ? 'error' : ''}
          />
          {errors.confirmPassword && (
            <span className="error-message">{errors.confirmPassword.message}</span>
          )}
        </div>
        
        {/* Age */}
        <div className="form-group">
          <label htmlFor="age">Age</label>
          <input
            id="age"
            type="number"
            {...register('age', {
              required: 'Age is required',
              min: {
                value: 18,
                message: 'You must be at least 18 years old'
              },
              max: {
                value: 120,
                message: 'Please enter a valid age'
              }
            })}
            placeholder="Enter age"
            className={errors.age ? 'error' : ''}
          />
          {errors.age && (
            <span className="error-message">{errors.age.message}</span>
          )}
        </div>
        
        {/* Terms Checkbox */}
        <div className="form-group checkbox">
          <input
            id="terms"
            type="checkbox"
            {...register('terms', {
              required: 'You must accept the terms and conditions'
            })}
          />
          <label htmlFor="terms">
            I accept the terms and conditions
          </label>
          {errors.terms && (
            <span className="error-message">{errors.terms.message}</span>
          )}
        </div>
        
        {/* Submit Button */}
        <button
          type="submit"
          disabled={isSubmitting || !isDirty || !isValid}
          className="submit-button"
        >
          {isSubmitting ? 'Signing Up...' : 'Sign Up'}
        </button>
      </form>
    </div>
  );
}

export default SignupForm;
```

---

## Advanced Features

### 1. Async Validation

**Check username availability:**

```jsx
const { register } = useForm();

<input
  {...register('username', {
    required: 'Username is required',
    validate: async (value) => {
      const response = await fetch(`/api/check-username?username=${value}`);
      const available = await response.json();
      return available || 'Username is taken';
    }
  })}
/>
```

---

### 2. Custom Validation

```jsx
<input
  {...register('password', {
    validate: {
      hasUpperCase: (value) =>
        /[A-Z]/.test(value) || 'Must contain uppercase letter',
      hasLowerCase: (value) =>
        /[a-z]/.test(value) || 'Must contain lowercase letter',
      hasNumber: (value) =>
        /\d/.test(value) || 'Must contain number',
      hasSpecialChar: (value) =>
        /[!@#$%^&*]/.test(value) || 'Must contain special character'
    }
  })}
/>
```

---

### 3. Dependent Fields

```jsx
const { register, watch } = useForm();

const country = watch('country');

return (
  <form>
    <select {...register('country')}>
      <option value="USA">USA</option>
      <option value="UK">UK</option>
    </select>
    
    {country === 'USA' && (
      <input {...register('state')} placeholder="State" />
    )}
    
    {country === 'UK' && (
      <input {...register('county')} placeholder="County" />
    )}
  </form>
);
```

---

### 4. Dynamic Fields (Field Array)

```jsx
import { useForm, useFieldArray } from 'react-hook-form';

function DynamicForm() {
  const { register, control } = useForm({
    defaultValues: {
      items: [{ name: '', quantity: 1 }]
    }
  });
  
  const { fields, append, remove } = useFieldArray({
    control,
    name: 'items'
  });
  
  return (
    <form>
      {fields.map((field, index) => (
        <div key={field.id}>
          <input {...register(`items.${index}.name`)} placeholder="Name" />
          <input
            type="number"
            {...register(`items.${index}.quantity`)}
            placeholder="Quantity"
          />
          <button type="button" onClick={() => remove(index)}>
            Remove
          </button>
        </div>
      ))}
      
      <button
        type="button"
        onClick={() => append({ name: '', quantity: 1 })}
      >
        Add Item
      </button>
    </form>
  );
}
```

---

### 5. Schema Validation (with Yup)

```jsx
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

const schema = yup.object().shape({
  email: yup.string().email().required(),
  password: yup.string().min(8).required(),
  age: yup.number().min(18).required()
});

function Form() {
  const {
    register,
    handleSubmit,
    formState: { errors }
  } = useForm({
    resolver: yupResolver(schema)
  });
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <p>{errors.email.message}</p>}
      
      <input type="password" {...register('password')} />
      {errors.password && <p>{errors.password.message}</p>}
      
      <input type="number" {...register('age')} />
      {errors.age && <p>{errors.age.message}</p>}
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Validation Modes

**Control when validation runs:**

```jsx
const { register } = useForm({
  mode: 'onSubmit'      // Validate on submit (default)
  // mode: 'onBlur'     // Validate on blur
  // mode: 'onChange'   // Validate on change
  // mode: 'onTouched'  // Validate on first blur, then on change
  // mode: 'all'        // Validate on blur and change
});
```

---

## Best Practices

**1. Use TypeScript for type safety**
```tsx
type FormData = {
  email: string;
  password: string;
};

const { register, handleSubmit } = useForm<FormData>();

const onSubmit = (data: FormData) => {
  console.log(data);
};
```

**2. Destructure only what you need**
```jsx
// ✅ Good
const { register, handleSubmit, formState: { errors } } = useForm();

// ❌ Avoid
const formMethods = useForm();
```

**3. Use validation mode wisely**
```jsx
// For better UX
const { register } = useForm({
  mode: 'onTouched'  // Validate after first blur
});
```

**4. Provide clear error messages**
```jsx
register('email', {
  required: 'Email is required',  // Clear message
  pattern: {
    value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
    message: 'Please enter a valid email address'  // User-friendly
  }
})
```

---

## React Hook Form vs Formik

| Feature | React Hook Form | Formik |
|---------|----------------|--------|
| **Bundle Size** | ~9KB | ~15KB |
| **Performance** | Excellent (minimal re-renders) | Good (more re-renders) |
| **Approach** | Uncontrolled | Controlled |
| **Learning Curve** | Easy | Moderate |
| **TypeScript** | Excellent | Good |
| **Schema Validation** | Via resolvers | Built-in (Yup) |
| **Field Arrays** | Built-in | Via FieldArray |
| **Popularity** | Growing fast | Very popular |

---

**Interview Tips:**
- React Hook Form = **performant form library**
- Uses **uncontrolled components** (refs)
- **Minimal re-renders** compared to controlled forms
- **~9KB** minified + gzipped
- **useForm()** hook is the main API
- **register()** to register inputs
- **handleSubmit()** for form submission
- **formState** for form state (errors, isValid, etc.)
- **watch()** to watch field values
- **setValue()** to set values programmatically
- Built-in **validation** (required, min, max, pattern)
- Custom **validate** function
- **Async validation** supported
- Schema validation with **Yup, Zod, Joi**
- **useFieldArray()** for dynamic fields
- Validation **modes**: onSubmit, onBlur, onChange, onTouched
- Better **performance** than Formik
- **TypeScript** support excellent
- Less code than traditional approach
- No **useState** needed for form values
- **DevTools** available
- Works with **UI libraries**
- **Default values** in useForm()
- **Reset** form with reset()
- **Dirty** = form modified
- **Touched** = field has been blurred
- Popular for **modern React apps**
- Recommended over **Formik** for new projects

</details>

---

### 72. What is Formik library?

<details>
<summary>View Answer</summary>

**Formik**

Formik is the **most popular** form library for React. It helps with **form state management**, **validation**, and **error handling** using a component-based approach.

**Official website:** https://formik.org/

**Created by:** Jared Palmer

---

## Why Use Formik?

**Problems Formik solves:**
- Managing form values
- Validation and error messages
- Handling form submission
- Reducing boilerplate code
- Form state (touched, dirty, submitting)

**Without Formik:**
```jsx
// Traditional React - lots of boilerplate
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [submitting, setSubmitting] = useState(false);
  
  const validate = () => {
    const errors = {};
    if (!email) errors.email = 'Required';
    if (!password) errors.password = 'Required';
    return errors;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const errors = validate();
    if (Object.keys(errors).length === 0) {
      // Submit form
    } else {
      setErrors(errors);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        onBlur={() => setTouched({ ...touched, email: true })}
      />
      {touched.email && errors.email && <span>{errors.email}</span>}
      {/* More boilerplate... */}
    </form>
  );
}
```

**With Formik:**
```jsx
import { Formik, Form, Field, ErrorMessage } from 'formik';

function LoginForm() {
  return (
    <Formik
      initialValues={{ email: '', password: '' }}
      validate={values => {
        const errors = {};
        if (!values.email) errors.email = 'Required';
        if (!values.password) errors.password = 'Required';
        return errors;
      }}
      onSubmit={(values, { setSubmitting }) => {
        console.log(values);
        setSubmitting(false);
      }}
    >
      <Form>
        <Field name="email" type="email" />
        <ErrorMessage name="email" component="div" />
        
        <Field name="password" type="password" />
        <ErrorMessage name="password" component="div" />
        
        <button type="submit">Submit</button>
      </Form>
    </Formik>
  );
}
```

---

## Installation

```bash
npm install formik
```

**With Yup (for schema validation):**
```bash
npm install formik yup
```

---

## Core Components

### 1. `<Formik>`

**The main wrapper component:**

```jsx
import { Formik } from 'formik';

<Formik
  initialValues={{ email: '', password: '' }}
  validate={values => { /* validation */ }}
  onSubmit={(values, actions) => { /* submit */ }}
>
  {/* form content */}
</Formik>
```

---

### 2. `<Form>`

**Wrapper for HTML form element:**

```jsx
import { Form } from 'formik';

<Form>
  {/* fields */}
</Form>

// Automatically handles onSubmit
// Equivalent to: <form onSubmit={formik.handleSubmit}>
```

---

### 3. `<Field>`

**Input field component:**

```jsx
import { Field } from 'formik';

<Field name="email" type="email" placeholder="Email" />

// Automatically handles:
// - value
// - onChange
// - onBlur
```

---

### 4. `<ErrorMessage>`

**Display validation errors:**

```jsx
import { ErrorMessage } from 'formik';

<ErrorMessage name="email" component="div" className="error" />

// Only shows if field is touched and has error
```

---

## Basic Example

```jsx
import React from 'react';
import { Formik, Form, Field, ErrorMessage } from 'formik';

function ContactForm() {
  const initialValues = {
    name: '',
    email: '',
    message: ''
  };
  
  const validate = (values) => {
    const errors = {};
    
    if (!values.name) {
      errors.name = 'Name is required';
    }
    
    if (!values.email) {
      errors.email = 'Email is required';
    } else if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(values.email)) {
      errors.email = 'Invalid email address';
    }
    
    if (!values.message) {
      errors.message = 'Message is required';
    } else if (values.message.length < 10) {
      errors.message = 'Message must be at least 10 characters';
    }
    
    return errors;
  };
  
  const onSubmit = (values, { setSubmitting, resetForm }) => {
    console.log('Form values:', values);
    
    // Simulate API call
    setTimeout(() => {
      alert(JSON.stringify(values, null, 2));
      setSubmitting(false);
      resetForm();
    }, 1000);
  };
  
  return (
    <div className="contact-form">
      <h1>Contact Us</h1>
      
      <Formik
        initialValues={initialValues}
        validate={validate}
        onSubmit={onSubmit}
      >
        {({ isSubmitting, errors, touched }) => (
          <Form>
            <div className="form-group">
              <label htmlFor="name">Name</label>
              <Field
                id="name"
                name="name"
                type="text"
                placeholder="Your name"
                className={touched.name && errors.name ? 'error' : ''}
              />
              <ErrorMessage name="name" component="div" className="error-message" />
            </div>
            
            <div className="form-group">
              <label htmlFor="email">Email</label>
              <Field
                id="email"
                name="email"
                type="email"
                placeholder="your.email@example.com"
                className={touched.email && errors.email ? 'error' : ''}
              />
              <ErrorMessage name="email" component="div" className="error-message" />
            </div>
            
            <div className="form-group">
              <label htmlFor="message">Message</label>
              <Field
                as="textarea"
                id="message"
                name="message"
                rows="5"
                placeholder="Your message"
                className={touched.message && errors.message ? 'error' : ''}
              />
              <ErrorMessage name="message" component="div" className="error-message" />
            </div>
            
            <button type="submit" disabled={isSubmitting}>
              {isSubmitting ? 'Sending...' : 'Send Message'}
            </button>
          </Form>
        )}
      </Formik>
    </div>
  );
}

export default ContactForm;
```

---

## Validation with Yup

**Schema-based validation (recommended):**

```jsx
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

const SignupSchema = Yup.object().shape({
  firstName: Yup.string()
    .min(2, 'Too short')
    .max(50, 'Too long')
    .required('Required'),
  lastName: Yup.string()
    .min(2, 'Too short')
    .max(50, 'Too long')
    .required('Required'),
  email: Yup.string()
    .email('Invalid email')
    .required('Required'),
  password: Yup.string()
    .min(8, 'Password must be at least 8 characters')
    .required('Required'),
  confirmPassword: Yup.string()
    .oneOf([Yup.ref('password'), null], 'Passwords must match')
    .required('Required')
});

function SignupForm() {
  return (
    <Formik
      initialValues={{
        firstName: '',
        lastName: '',
        email: '',
        password: '',
        confirmPassword: ''
      }}
      validationSchema={SignupSchema}
      onSubmit={(values) => {
        console.log(values);
      }}
    >
      {({ errors, touched }) => (
        <Form>
          <Field name="firstName" />
          {errors.firstName && touched.firstName && <div>{errors.firstName}</div>}
          
          <Field name="lastName" />
          {errors.lastName && touched.lastName && <div>{errors.lastName}</div>}
          
          <Field name="email" type="email" />
          {errors.email && touched.email && <div>{errors.email}</div>}
          
          <Field name="password" type="password" />
          {errors.password && touched.password && <div>{errors.password}</div>}
          
          <Field name="confirmPassword" type="password" />
          {errors.confirmPassword && touched.confirmPassword && (
            <div>{errors.confirmPassword}</div>
          )}
          
          <button type="submit">Sign Up</button>
        </Form>
      )}
    </Formik>
  );
}
```

---

## useFormik Hook

**Alternative to component-based approach:**

```jsx
import { useFormik } from 'formik';

function LoginForm() {
  const formik = useFormik({
    initialValues: {
      email: '',
      password: ''
    },
    validate: values => {
      const errors = {};
      if (!values.email) errors.email = 'Required';
      if (!values.password) errors.password = 'Required';
      return errors;
    },
    onSubmit: values => {
      console.log(values);
    }
  });
  
  return (
    <form onSubmit={formik.handleSubmit}>
      <input
        id="email"
        name="email"
        type="email"
        onChange={formik.handleChange}
        onBlur={formik.handleBlur}
        value={formik.values.email}
      />
      {formik.touched.email && formik.errors.email && (
        <div>{formik.errors.email}</div>
      )}
      
      <input
        id="password"
        name="password"
        type="password"
        onChange={formik.handleChange}
        onBlur={formik.handleBlur}
        value={formik.values.password}
      />
      {formik.touched.password && formik.errors.password && (
        <div>{formik.errors.password}</div>
      )}
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Field Types

### Text Input

```jsx
<Field name="username" type="text" placeholder="Username" />
```

### Email Input

```jsx
<Field name="email" type="email" placeholder="Email" />
```

### Password Input

```jsx
<Field name="password" type="password" placeholder="Password" />
```

### Textarea

```jsx
<Field as="textarea" name="message" rows="5" />
```

### Select Dropdown

```jsx
<Field as="select" name="country">
  <option value="">Select a country</option>
  <option value="USA">USA</option>
  <option value="UK">UK</option>
  <option value="Canada">Canada</option>
</Field>
```

### Checkbox

```jsx
<label>
  <Field type="checkbox" name="terms" />
  I accept the terms and conditions
</label>
```

### Radio Buttons

```jsx
<div>
  <label>
    <Field type="radio" name="gender" value="male" />
    Male
  </label>
  <label>
    <Field type="radio" name="gender" value="female" />
    Female
  </label>
</div>
```

---

## FieldArray

**Handle dynamic form fields:**

```jsx
import { Formik, Form, Field, FieldArray } from 'formik';

function TodoList() {
  return (
    <Formik
      initialValues={{
        todos: ['', '']
      }}
      onSubmit={values => {
        console.log(values);
      }}
    >
      {({ values }) => (
        <Form>
          <FieldArray name="todos">
            {({ push, remove }) => (
              <div>
                {values.todos.map((todo, index) => (
                  <div key={index}>
                    <Field name={`todos.${index}`} placeholder="Todo" />
                    <button type="button" onClick={() => remove(index)}>
                      Remove
                    </button>
                  </div>
                ))}
                <button type="button" onClick={() => push('')}>
                  Add Todo
                </button>
              </div>
            )}
          </FieldArray>
          <button type="submit">Submit</button>
        </Form>
      )}
    </Formik>
  );
}
```

---

## Custom Field Component

```jsx
import { useField } from 'formik';

function MyTextField({ label, ...props }) {
  const [field, meta] = useField(props);
  
  return (
    <div className="form-group">
      <label htmlFor={props.id || props.name}>{label}</label>
      <input {...field} {...props} className="form-control" />
      {meta.touched && meta.error ? (
        <div className="error">{meta.error}</div>
      ) : null}
    </div>
  );
}

// Usage
<Formik
  initialValues={{ email: '' }}
  onSubmit={values => console.log(values)}
>
  <Form>
    <MyTextField label="Email" name="email" type="email" />
    <button type="submit">Submit</button>
  </Form>
</Formik>
```

---

## Form State

**Access form state:**

```jsx
<Formik
  initialValues={{ email: '' }}
  onSubmit={values => console.log(values)}
>
  {({
    values,        // Current form values
    errors,        // Validation errors
    touched,       // Fields that have been touched
    isSubmitting,  // Is form submitting?
    isValid,       // Is form valid?
    dirty,         // Has form been modified?
    handleSubmit,
    handleReset
  }) => (
    <Form>
      <Field name="email" />
      
      <div>
        <p>Current value: {values.email}</p>
        <p>Is dirty: {dirty ? 'Yes' : 'No'}</p>
        <p>Is valid: {isValid ? 'Yes' : 'No'}</p>
      </div>
      
      <button type="submit" disabled={isSubmitting || !isValid}>
        Submit
      </button>
      <button type="button" onClick={handleReset}>
        Reset
      </button>
    </Form>
  )}
</Formik>
```

---

## Async Validation

```jsx
const validateUsername = async (value) => {
  const response = await fetch(`/api/check-username?username=${value}`);
  const available = await response.json();
  
  if (!available) {
    return 'Username is taken';
  }
};

<Formik
  initialValues={{ username: '' }}
  validate={async (values) => {
    const errors = {};
    const usernameError = await validateUsername(values.username);
    if (usernameError) {
      errors.username = usernameError;
    }
    return errors;
  }}
  onSubmit={values => console.log(values)}
>
  {/* form */}
</Formik>
```

---

## Complete Example: Multi-field Form

```jsx
import React from 'react';
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

const ProfileSchema = Yup.object().shape({
  firstName: Yup.string().required('Required'),
  lastName: Yup.string().required('Required'),
  email: Yup.string().email('Invalid email').required('Required'),
  age: Yup.number().min(18, 'Must be 18+').required('Required'),
  gender: Yup.string().required('Required'),
  country: Yup.string().required('Required'),
  bio: Yup.string().min(10, 'Min 10 characters').required('Required'),
  terms: Yup.boolean().oneOf([true], 'Must accept terms')
});

function ProfileForm() {
  return (
    <div className="profile-form">
      <h1>Create Profile</h1>
      
      <Formik
        initialValues={{
          firstName: '',
          lastName: '',
          email: '',
          age: '',
          gender: '',
          country: '',
          bio: '',
          terms: false
        }}
        validationSchema={ProfileSchema}
        onSubmit={(values, { setSubmitting }) => {
          setTimeout(() => {
            alert(JSON.stringify(values, null, 2));
            setSubmitting(false);
          }, 1000);
        }}
      >
        {({ isSubmitting, isValid, dirty }) => (
          <Form>
            <div className="form-row">
              <div className="form-group">
                <label>First Name</label>
                <Field name="firstName" />
                <ErrorMessage name="firstName" component="div" className="error" />
              </div>
              
              <div className="form-group">
                <label>Last Name</label>
                <Field name="lastName" />
                <ErrorMessage name="lastName" component="div" className="error" />
              </div>
            </div>
            
            <div className="form-group">
              <label>Email</label>
              <Field name="email" type="email" />
              <ErrorMessage name="email" component="div" className="error" />
            </div>
            
            <div className="form-group">
              <label>Age</label>
              <Field name="age" type="number" />
              <ErrorMessage name="age" component="div" className="error" />
            </div>
            
            <div className="form-group">
              <label>Gender</label>
              <Field as="select" name="gender">
                <option value="">Select gender</option>
                <option value="male">Male</option>
                <option value="female">Female</option>
                <option value="other">Other</option>
              </Field>
              <ErrorMessage name="gender" component="div" className="error" />
            </div>
            
            <div className="form-group">
              <label>Country</label>
              <Field as="select" name="country">
                <option value="">Select country</option>
                <option value="USA">USA</option>
                <option value="UK">UK</option>
                <option value="Canada">Canada</option>
              </Field>
              <ErrorMessage name="country" component="div" className="error" />
            </div>
            
            <div className="form-group">
              <label>Bio</label>
              <Field as="textarea" name="bio" rows="5" />
              <ErrorMessage name="bio" component="div" className="error" />
            </div>
            
            <div className="form-group checkbox">
              <label>
                <Field type="checkbox" name="terms" />
                I accept the terms and conditions
              </label>
              <ErrorMessage name="terms" component="div" className="error" />
            </div>
            
            <button
              type="submit"
              disabled={isSubmitting || !isValid || !dirty}
            >
              {isSubmitting ? 'Creating Profile...' : 'Create Profile'}
            </button>
          </Form>
        )}
      </Formik>
    </div>
  );
}

export default ProfileForm;
```

---

## Best Practices

**1. Use Yup for validation**
```jsx
// ✅ Good - schema validation
validationSchema={YupSchema}

// ❌ Avoid - manual validation (error-prone)
validate={values => { /* manual validation */ }}
```

**2. Use ErrorMessage component**
```jsx
<ErrorMessage name="email" component="div" className="error" />
```

**3. Disable submit when invalid**
```jsx
<button type="submit" disabled={isSubmitting || !isValid}>
  Submit
</button>
```

**4. Show errors only when touched**
```jsx
{touched.email && errors.email && <div>{errors.email}</div>}
```

**5. Use TypeScript**
```tsx
interface FormValues {
  email: string;
  password: string;
}

<Formik<FormValues>
  initialValues={{ email: '', password: '' }}
  onSubmit={(values: FormValues) => { /* ... */ }}
>
```

---

**Interview Tips:**
- Formik = **most popular form library** for React
- Handles **form state**, **validation**, **submission**
- Uses **controlled components** approach
- **~15KB** minified + gzipped
- **`<Formik>`** wrapper component
- **`<Form>`** replaces HTML form
- **`<Field>`** for input fields
- **`<ErrorMessage>`** to display errors
- **`<FieldArray>`** for dynamic fields
- **initialValues** prop sets initial form values
- **validate** function for custom validation
- **validationSchema** for Yup validation
- **onSubmit** handler receives values and actions
- **useFormik()** hook alternative
- **useField()** for custom components
- **Render props** pattern for accessing form state
- **values** = current form values
- **errors** = validation errors
- **touched** = fields that have been blurred
- **isSubmitting** = is form being submitted
- **isValid** = is form valid
- **dirty** = has form been modified
- Works great with **Yup** validation
- **Async validation** supported
- **Field-level** and **form-level** validation
- Created by **Jared Palmer**
- Very **popular** (2M+ weekly downloads)
- More **re-renders** than React Hook Form
- Easier **learning curve** than React Hook Form

</details>

---

### 73. How do you implement form validation in React?

<details>
<summary>View Answer</summary>

**Form Validation in React**

Form validation can be implemented in React using several approaches:
1. **Built-in HTML5 validation**
2. **Manual validation with state**
3. **Third-party libraries** (React Hook Form, Formik)
4. **Schema validation** (Yup, Zod)

---

## 1. Built-in HTML5 Validation

**Using native HTML attributes:**

```jsx
function SimpleForm() {
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Check if form is valid
    if (e.target.checkValidity()) {
      const formData = new FormData(e.target);
      console.log(Object.fromEntries(formData));
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        type="email"
        required
        placeholder="Email"
      />
      
      <input
        name="password"
        type="password"
        required
        minLength={8}
        placeholder="Password"
      />
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

**HTML5 validation attributes:**
- `required` - Field is required
- `minLength` / `maxLength` - String length
- `min` / `max` - Number range
- `pattern` - Regex pattern
- `type` - Input type (email, number, url, etc.)

**Limitations:**
- ❌ Limited customization
- ❌ Browser-dependent UI
- ❌ Hard to style
- ❌ Limited error messages

---

## 2. Manual Validation with State

**Custom validation logic:**

```jsx
import React, { useState } from 'react';

function SignupForm() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  });
  
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  // Validate individual field
  const validateField = (name, value) => {
    switch (name) {
      case 'username':
        if (!value) return 'Username is required';
        if (value.length < 3) return 'Username must be at least 3 characters';
        if (!/^[a-zA-Z0-9_]+$/.test(value)) return 'Username can only contain letters, numbers, and underscores';
        return '';
        
      case 'email':
        if (!value) return 'Email is required';
        if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(value)) return 'Invalid email address';
        return '';
        
      case 'password':
        if (!value) return 'Password is required';
        if (value.length < 8) return 'Password must be at least 8 characters';
        if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value)) return 'Password must contain uppercase, lowercase, and number';
        return '';
        
      case 'confirmPassword':
        if (!value) return 'Please confirm your password';
        if (value !== formData.password) return 'Passwords do not match';
        return '';
        
      default:
        return '';
    }
  };
  
  // Validate all fields
  const validateForm = () => {
    const newErrors = {};
    
    Object.keys(formData).forEach(field => {
      const error = validateField(field, formData[field]);
      if (error) newErrors[field] = error;
    });
    
    return newErrors;
  };
  
  // Handle input change
  const handleChange = (e) => {
    const { name, value } = e.target;
    
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
    
    // Validate field if already touched
    if (touched[name]) {
      const error = validateField(name, value);
      setErrors(prev => ({
        ...prev,
        [name]: error
      }));
    }
  };
  
  // Handle input blur
  const handleBlur = (e) => {
    const { name, value } = e.target;
    
    setTouched(prev => ({
      ...prev,
      [name]: true
    }));
    
    const error = validateField(name, value);
    setErrors(prev => ({
      ...prev,
      [name]: error
    }));
  };
  
  // Handle form submission
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Mark all fields as touched
    const allTouched = Object.keys(formData).reduce((acc, field) => {
      acc[field] = true;
      return acc;
    }, {});
    setTouched(allTouched);
    
    // Validate all fields
    const newErrors = validateForm();
    setErrors(newErrors);
    
    // Submit if no errors
    if (Object.keys(newErrors).length === 0) {
      console.log('Form submitted:', formData);
      // Submit to API
    }
  };
  
  return (
    <form onSubmit={handleSubmit} className="signup-form">
      <h1>Sign Up</h1>
      
      {/* Username */}
      <div className="form-group">
        <label htmlFor="username">Username</label>
        <input
          id="username"
          name="username"
          type="text"
          value={formData.username}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.username && errors.username ? 'error' : ''}
        />
        {touched.username && errors.username && (
          <span className="error-message">{errors.username}</span>
        )}
      </div>
      
      {/* Email */}
      <div className="form-group">
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.email && errors.email ? 'error' : ''}
        />
        {touched.email && errors.email && (
          <span className="error-message">{errors.email}</span>
        )}
      </div>
      
      {/* Password */}
      <div className="form-group">
        <label htmlFor="password">Password</label>
        <input
          id="password"
          name="password"
          type="password"
          value={formData.password}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.password && errors.password ? 'error' : ''}
        />
        {touched.password && errors.password && (
          <span className="error-message">{errors.password}</span>
        )}
      </div>
      
      {/* Confirm Password */}
      <div className="form-group">
        <label htmlFor="confirmPassword">Confirm Password</label>
        <input
          id="confirmPassword"
          name="confirmPassword"
          type="password"
          value={formData.confirmPassword}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.confirmPassword && errors.confirmPassword ? 'error' : ''}
        />
        {touched.confirmPassword && errors.confirmPassword && (
          <span className="error-message">{errors.confirmPassword}</span>
        )}
      </div>
      
      <button type="submit">Sign Up</button>
    </form>
  );
}

export default SignupForm;
```

---

## 3. With React Hook Form

**Performant validation with minimal code:**

```jsx
import { useForm } from 'react-hook-form';

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors }
  } = useForm({
    mode: 'onBlur'  // Validate on blur
  });
  
  const onSubmit = (data) => {
    console.log(data);
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input
          {...register('email', {
            required: 'Email is required',
            pattern: {
              value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
              message: 'Invalid email address'
            }
          })}
          placeholder="Email"
        />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      <div>
        <input
          type="password"
          {...register('password', {
            required: 'Password is required',
            minLength: {
              value: 8,
              message: 'Password must be at least 8 characters'
            },
            validate: {
              hasUpperCase: (value) =>
                /[A-Z]/.test(value) || 'Must contain uppercase letter',
              hasLowerCase: (value) =>
                /[a-z]/.test(value) || 'Must contain lowercase letter',
              hasNumber: (value) =>
                /\d/.test(value) || 'Must contain number'
            }
          })}
          placeholder="Password"
        />
        {errors.password && <span>{errors.password.message}</span>}
      </div>
      
      <button type="submit">Login</button>
    </form>
  );
}
```

---

## 4. With Formik

**Component-based validation:**

```jsx
import { Formik, Form, Field, ErrorMessage } from 'formik';

function ContactForm() {
  const validate = (values) => {
    const errors = {};
    
    // Name validation
    if (!values.name) {
      errors.name = 'Name is required';
    } else if (values.name.length < 2) {
      errors.name = 'Name must be at least 2 characters';
    }
    
    // Email validation
    if (!values.email) {
      errors.email = 'Email is required';
    } else if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(values.email)) {
      errors.email = 'Invalid email address';
    }
    
    // Message validation
    if (!values.message) {
      errors.message = 'Message is required';
    } else if (values.message.length < 10) {
      errors.message = 'Message must be at least 10 characters';
    }
    
    return errors;
  };
  
  return (
    <Formik
      initialValues={{ name: '', email: '', message: '' }}
      validate={validate}
      onSubmit={(values) => {
        console.log(values);
      }}
    >
      {({ errors, touched }) => (
        <Form>
          <div>
            <Field name="name" placeholder="Name" />
            {errors.name && touched.name && <div>{errors.name}</div>}
          </div>
          
          <div>
            <Field name="email" type="email" placeholder="Email" />
            {errors.email && touched.email && <div>{errors.email}</div>}
          </div>
          
          <div>
            <Field as="textarea" name="message" placeholder="Message" />
            {errors.message && touched.message && <div>{errors.message}</div>}
          </div>
          
          <button type="submit">Submit</button>
        </Form>
      )}
    </Formik>
  );
}
```

---

## 5. Schema Validation with Yup

**Declarative validation schema:**

```jsx
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

// Define validation schema
const schema = yup.object().shape({
  username: yup
    .string()
    .required('Username is required')
    .min(3, 'Username must be at least 3 characters')
    .max(20, 'Username must be less than 20 characters')
    .matches(/^[a-zA-Z0-9_]+$/, 'Only letters, numbers, and underscores'),
  
  email: yup
    .string()
    .required('Email is required')
    .email('Invalid email address'),
  
  password: yup
    .string()
    .required('Password is required')
    .min(8, 'Password must be at least 8 characters')
    .matches(/[a-z]/, 'Must contain lowercase letter')
    .matches(/[A-Z]/, 'Must contain uppercase letter')
    .matches(/\d/, 'Must contain number'),
  
  confirmPassword: yup
    .string()
    .required('Please confirm your password')
    .oneOf([yup.ref('password'), null], 'Passwords must match'),
  
  age: yup
    .number()
    .required('Age is required')
    .min(18, 'Must be 18 or older')
    .max(120, 'Invalid age'),
  
  terms: yup
    .boolean()
    .oneOf([true], 'You must accept the terms')
});

function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors }
  } = useForm({
    resolver: yupResolver(schema)
  });
  
  const onSubmit = (data) => {
    console.log(data);
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('username')} placeholder="Username" />
      {errors.username && <span>{errors.username.message}</span>}
      
      <input {...register('email')} placeholder="Email" />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input type="password" {...register('password')} placeholder="Password" />
      {errors.password && <span>{errors.password.message}</span>}
      
      <input type="password" {...register('confirmPassword')} placeholder="Confirm" />
      {errors.confirmPassword && <span>{errors.confirmPassword.message}</span>}
      
      <input type="number" {...register('age')} placeholder="Age" />
      {errors.age && <span>{errors.age.message}</span>}
      
      <label>
        <input type="checkbox" {...register('terms')} />
        I accept the terms
      </label>
      {errors.terms && <span>{errors.terms.message}</span>}
      
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

---

## 6. Async Validation

**Check availability (username, email):**

```jsx
import { useForm } from 'react-hook-form';

function UsernameForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();
  
  const checkUsernameAvailability = async (username) => {
    // Simulate API call
    const response = await fetch(`/api/check-username?username=${username}`);
    const data = await response.json();
    return data.available || 'Username is already taken';
  };
  
  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <input
        {...register('username', {
          required: 'Username is required',
          validate: checkUsernameAvailability
        })}
        placeholder="Username"
      />
      {errors.username && <span>{errors.username.message}</span>}
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

**With debouncing:**

```jsx
import { useForm } from 'react-hook-form';
import { useState } from 'react';

function UsernameFormDebounced() {
  const { register, handleSubmit, formState: { errors } } = useForm();
  const [checking, setChecking] = useState(false);
  
  // Debounced validation
  let timeout;
  const checkUsername = async (username) => {
    setChecking(true);
    
    return new Promise((resolve) => {
      clearTimeout(timeout);
      timeout = setTimeout(async () => {
        const response = await fetch(`/api/check-username?username=${username}`);
        const data = await response.json();
        setChecking(false);
        resolve(data.available || 'Username is taken');
      }, 500);  // 500ms debounce
    });
  };
  
  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <div>
        <input
          {...register('username', {
            required: 'Username is required',
            validate: checkUsername
          })}
          placeholder="Username"
        />
        {checking && <span>Checking availability...</span>}
        {errors.username && <span>{errors.username.message}</span>}
      </div>
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## 7. Real-time Validation

**Validate as user types:**

```jsx
import { useState, useEffect } from 'react';

function PasswordStrength() {
  const [password, setPassword] = useState('');
  const [strength, setStrength] = useState({
    score: 0,
    feedback: []
  });
  
  useEffect(() => {
    const feedback = [];
    let score = 0;
    
    if (password.length >= 8) {
      score++;
      feedback.push('✓ At least 8 characters');
    } else {
      feedback.push('✗ At least 8 characters');
    }
    
    if (/[a-z]/.test(password)) {
      score++;
      feedback.push('✓ Lowercase letter');
    } else {
      feedback.push('✗ Lowercase letter');
    }
    
    if (/[A-Z]/.test(password)) {
      score++;
      feedback.push('✓ Uppercase letter');
    } else {
      feedback.push('✗ Uppercase letter');
    }
    
    if (/\d/.test(password)) {
      score++;
      feedback.push('✓ Number');
    } else {
      feedback.push('✗ Number');
    }
    
    if (/[!@#$%^&*]/.test(password)) {
      score++;
      feedback.push('✓ Special character');
    } else {
      feedback.push('✗ Special character');
    }
    
    setStrength({ score, feedback });
  }, [password]);
  
  const getStrengthColor = () => {
    if (strength.score <= 1) return 'red';
    if (strength.score <= 3) return 'orange';
    return 'green';
  };
  
  return (
    <div>
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      
      <div className="strength-meter">
        <div
          className="strength-bar"
          style={{
            width: `${(strength.score / 5) * 100}%`,
            backgroundColor: getStrengthColor()
          }}
        />
      </div>
      
      <ul className="feedback">
        {strength.feedback.map((item, index) => (
          <li key={index} style={{ color: item.startsWith('✓') ? 'green' : 'red' }}>
            {item}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 8. Custom Validation Hook

**Reusable validation logic:**

```jsx
import { useState } from 'react';

function useFormValidation(initialState, validate) {
  const [values, setValues] = useState(initialState);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Clear error when user starts typing
    if (touched[name]) {
      const fieldErrors = validate({ ...values, [name]: value });
      setErrors(prev => ({ ...prev, [name]: fieldErrors[name] }));
    }
  };
  
  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    
    const fieldErrors = validate(values);
    setErrors(prev => ({ ...prev, [name]: fieldErrors[name] }));
  };
  
  const handleSubmit = (onSubmit) => async (e) => {
    e.preventDefault();
    
    const allTouched = Object.keys(values).reduce((acc, key) => {
      acc[key] = true;
      return acc;
    }, {});
    setTouched(allTouched);
    
    const validationErrors = validate(values);
    setErrors(validationErrors);
    
    if (Object.keys(validationErrors).length === 0) {
      setIsSubmitting(true);
      await onSubmit(values);
      setIsSubmitting(false);
    }
  };
  
  const reset = () => {
    setValues(initialState);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  };
  
  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    reset
  };
}

// Usage
function LoginForm() {
  const validate = (values) => {
    const errors = {};
    
    if (!values.email) {
      errors.email = 'Email is required';
    } else if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(values.email)) {
      errors.email = 'Invalid email';
    }
    
    if (!values.password) {
      errors.password = 'Password is required';
    } else if (values.password.length < 8) {
      errors.password = 'Password must be at least 8 characters';
    }
    
    return errors;
  };
  
  const {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit
  } = useFormValidation({ email: '', password: '' }, validate);
  
  const onSubmit = async (values) => {
    console.log('Submitting:', values);
    // API call
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input
          name="email"
          type="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.email && errors.email && <span>{errors.email}</span>}
      </div>
      
      <div>
        <input
          name="password"
          type="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.password && errors.password && <span>{errors.password}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

---

## Validation Approaches Comparison

| Approach | Pros | Cons | Best For |
|----------|------|------|----------|
| **HTML5** | Native, simple | Limited customization | Simple forms |
| **Manual State** | Full control | Lots of boilerplate | Learning, custom needs |
| **React Hook Form** | Performant, minimal re-renders | Learning curve | Complex forms |
| **Formik** | Easy to use, popular | More re-renders | Rapid development |
| **Yup/Zod** | Declarative, reusable | Additional dependency | Schema validation |

---

## Best Practices

**1. Validate on blur, show errors on touch**
```jsx
// ✅ Good UX
<input
  onBlur={handleBlur}
  // Show error only after user touches field
/>
{touched.email && errors.email && <span>{errors.email}</span>}
```

**2. Clear error messages**
```jsx
// ✅ Clear
errors.email = 'Please enter a valid email address'

// ❌ Unclear
errors.email = 'Invalid'
```

**3. Disable submit when invalid**
```jsx
<button type="submit" disabled={isSubmitting || !isValid}>
  Submit
</button>
```

**4. Show validation state visually**
```jsx
<input
  className={`
    ${touched.email && errors.email ? 'invalid' : ''}
    ${touched.email && !errors.email ? 'valid' : ''}
  `}
/>
```

**5. Use schema validation for complex forms**
```jsx
// ✅ Maintainable
const schema = yup.object().shape({
  email: yup.string().email().required(),
  age: yup.number().min(18).required()
});
```

**6. Debounce async validation**
```jsx
// Avoid too many API calls
const checkUsername = debounce(async (username) => {
  // API call
}, 500);
```

**7. Provide real-time feedback**
```jsx
// Password strength meter
// Character counter
// Availability checker
```

---

**Interview Tips:**
- Form validation = **checking user input** before submission
- **Client-side** validation (UX) vs **server-side** validation (security)
- Approaches: **HTML5**, **manual state**, **libraries** (React Hook Form, Formik)
- **HTML5 validation**: native attributes (required, minLength, pattern)
- **Manual validation**: useState for values, errors, touched
- **touched** = field has been blurred
- **dirty** = field has been modified
- **errors** = validation error messages
- Validate on **blur** for better UX
- Show errors only after field is **touched**
- **React Hook Form**: performant, minimal re-renders
- **Formik**: popular, component-based
- **Schema validation**: Yup, Zod for declarative rules
- **Async validation**: check username/email availability
- **Debounce** async validation to reduce API calls
- **Real-time validation**: password strength, character count
- **Custom hooks** for reusable validation logic
- Always **disable submit** when form is invalid
- Provide **clear error messages**
- Use **visual feedback** (red border, green checkmark)
- **Client + Server** validation for security
- **Yup** = most popular schema validator
- **Zod** = TypeScript-first alternative
- **validate** function returns errors object
- **onSubmit** only called if form is valid
- Form libraries handle **state management** automatically
- **Field-level** vs **form-level** validation
- **Synchronous** vs **asynchronous** validation
- **validateOnChange** vs **validateOnBlur** vs **validateOnSubmit**
- Best UX: validate on blur, show on touch
- **Progressive enhancement**: HTML5 + JavaScript validation

</details>

---

### 74. What is Yup validation library?

<details>
<summary>View Answer</summary>

**Yup**

Yup is a **JavaScript schema validation library** for validating object shapes. It's commonly used with **Formik** and **React Hook Form** for form validation.

**Official website:** https://github.com/jquense/yup

**Created by:** Jason Quense

---

## Why Use Yup?

**Problems with manual validation:**
```jsx
// ❌ Manual validation - repetitive and error-prone
const validate = (values) => {
  const errors = {};
  
  if (!values.email) {
    errors.email = 'Email is required';
  } else if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(values.email)) {
    errors.email = 'Invalid email';
  }
  
  if (!values.age) {
    errors.age = 'Age is required';
  } else if (values.age < 18) {
    errors.age = 'Must be 18 or older';
  }
  
  // ... more validation
  return errors;
};
```

**With Yup:**
```jsx
import * as yup from 'yup';

// ✅ Declarative schema - clean and maintainable
const schema = yup.object().shape({
  email: yup
    .string()
    .email('Invalid email')
    .required('Email is required'),
  
  age: yup
    .number()
    .min(18, 'Must be 18 or older')
    .required('Age is required')
});
```

---

## Benefits

1. **Declarative** - Define validation rules as schema
2. **Reusable** - Share schemas across components
3. **Type-safe** - TypeScript support
4. **Composable** - Build complex schemas from simple ones
5. **Async validation** - Built-in async support
6. **Error messages** - Customizable error messages
7. **Popular** - Works with Formik, React Hook Form

---

## Installation

```bash
npm install yup
```

**With React Hook Form:**
```bash
npm install yup @hookform/resolvers
```

**With Formik:**
```bash
npm install yup formik
```

---

## Basic Usage

### String Validation

```jsx
import * as yup from 'yup';

const schema = yup.object().shape({
  name: yup.string().required('Name is required'),
  
  email: yup
    .string()
    .email('Invalid email address')
    .required('Email is required'),
  
  username: yup
    .string()
    .min(3, 'Username must be at least 3 characters')
    .max(20, 'Username must be less than 20 characters')
    .matches(/^[a-zA-Z0-9_]+$/, 'Only letters, numbers, and underscores')
    .required('Username is required'),
  
  url: yup
    .string()
    .url('Must be a valid URL')
    .required('URL is required')
});
```

---

### Number Validation

```jsx
const schema = yup.object().shape({
  age: yup
    .number()
    .typeError('Age must be a number')
    .min(18, 'Must be at least 18')
    .max(120, 'Must be less than 120')
    .required('Age is required'),
  
  price: yup
    .number()
    .positive('Price must be positive')
    .required('Price is required'),
  
  quantity: yup
    .number()
    .integer('Quantity must be an integer')
    .positive('Quantity must be positive')
    .required('Quantity is required')
});
```

---

### Boolean Validation

```jsx
const schema = yup.object().shape({
  terms: yup
    .boolean()
    .oneOf([true], 'You must accept the terms and conditions')
    .required('Required'),
  
  newsletter: yup.boolean()
});
```

---

### Date Validation

```jsx
const schema = yup.object().shape({
  birthDate: yup
    .date()
    .max(new Date(), 'Birth date cannot be in the future')
    .required('Birth date is required'),
  
  appointmentDate: yup
    .date()
    .min(new Date(), 'Appointment must be in the future')
    .required('Appointment date is required')
});
```

---

### Array Validation

```jsx
const schema = yup.object().shape({
  tags: yup
    .array()
    .of(yup.string())
    .min(1, 'At least one tag is required')
    .max(5, 'Maximum 5 tags allowed')
    .required('Tags are required'),
  
  hobbies: yup
    .array()
    .of(yup.string().min(2))
    .required('At least one hobby is required')
});
```

---

### Object Validation

```jsx
const schema = yup.object().shape({
  address: yup.object().shape({
    street: yup.string().required('Street is required'),
    city: yup.string().required('City is required'),
    zipCode: yup
      .string()
      .matches(/^\d{5}$/, 'Invalid zip code')
      .required('Zip code is required')
  })
});
```

---

## With React Hook Form

```jsx
import React from 'react';
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

// Define validation schema
const schema = yup.object().shape({
  firstName: yup
    .string()
    .required('First name is required')
    .min(2, 'First name must be at least 2 characters'),
  
  lastName: yup
    .string()
    .required('Last name is required')
    .min(2, 'Last name must be at least 2 characters'),
  
  email: yup
    .string()
    .email('Invalid email address')
    .required('Email is required'),
  
  password: yup
    .string()
    .required('Password is required')
    .min(8, 'Password must be at least 8 characters')
    .matches(/[a-z]/, 'Password must contain at least one lowercase letter')
    .matches(/[A-Z]/, 'Password must contain at least one uppercase letter')
    .matches(/\d/, 'Password must contain at least one number'),
  
  confirmPassword: yup
    .string()
    .oneOf([yup.ref('password'), null], 'Passwords must match')
    .required('Please confirm your password'),
  
  age: yup
    .number()
    .typeError('Age must be a number')
    .min(18, 'You must be at least 18 years old')
    .max(120, 'Please enter a valid age')
    .required('Age is required'),
  
  terms: yup
    .boolean()
    .oneOf([true], 'You must accept the terms and conditions')
});

function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting }
  } = useForm({
    resolver: yupResolver(schema),
    mode: 'onBlur'
  });
  
  const onSubmit = async (data) => {
    console.log('Form Data:', data);
    // API call here
  };
  
  return (
    <div className="signup-form">
      <h1>Sign Up</h1>
      
      <form onSubmit={handleSubmit(onSubmit)}>
        {/* First Name */}
        <div className="form-group">
          <label htmlFor="firstName">First Name</label>
          <input
            id="firstName"
            {...register('firstName')}
            className={errors.firstName ? 'error' : ''}
          />
          {errors.firstName && (
            <span className="error-message">{errors.firstName.message}</span>
          )}
        </div>
        
        {/* Last Name */}
        <div className="form-group">
          <label htmlFor="lastName">Last Name</label>
          <input
            id="lastName"
            {...register('lastName')}
            className={errors.lastName ? 'error' : ''}
          />
          {errors.lastName && (
            <span className="error-message">{errors.lastName.message}</span>
          )}
        </div>
        
        {/* Email */}
        <div className="form-group">
          <label htmlFor="email">Email</label>
          <input
            id="email"
            type="email"
            {...register('email')}
            className={errors.email ? 'error' : ''}
          />
          {errors.email && (
            <span className="error-message">{errors.email.message}</span>
          )}
        </div>
        
        {/* Password */}
        <div className="form-group">
          <label htmlFor="password">Password</label>
          <input
            id="password"
            type="password"
            {...register('password')}
            className={errors.password ? 'error' : ''}
          />
          {errors.password && (
            <span className="error-message">{errors.password.message}</span>
          )}
        </div>
        
        {/* Confirm Password */}
        <div className="form-group">
          <label htmlFor="confirmPassword">Confirm Password</label>
          <input
            id="confirmPassword"
            type="password"
            {...register('confirmPassword')}
            className={errors.confirmPassword ? 'error' : ''}
          />
          {errors.confirmPassword && (
            <span className="error-message">{errors.confirmPassword.message}</span>
          )}
        </div>
        
        {/* Age */}
        <div className="form-group">
          <label htmlFor="age">Age</label>
          <input
            id="age"
            type="number"
            {...register('age')}
            className={errors.age ? 'error' : ''}
          />
          {errors.age && (
            <span className="error-message">{errors.age.message}</span>
          )}
        </div>
        
        {/* Terms */}
        <div className="form-group checkbox">
          <label>
            <input type="checkbox" {...register('terms')} />
            I accept the terms and conditions
          </label>
          {errors.terms && (
            <span className="error-message">{errors.terms.message}</span>
          )}
        </div>
        
        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Signing Up...' : 'Sign Up'}
        </button>
      </form>
    </div>
  );
}

export default SignupForm;
```

---

## With Formik

```jsx
import React from 'react';
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as yup from 'yup';

const validationSchema = yup.object().shape({
  name: yup
    .string()
    .required('Name is required')
    .min(2, 'Name must be at least 2 characters'),
  
  email: yup
    .string()
    .email('Invalid email address')
    .required('Email is required'),
  
  message: yup
    .string()
    .required('Message is required')
    .min(10, 'Message must be at least 10 characters')
    .max(500, 'Message must be less than 500 characters')
});

function ContactForm() {
  const initialValues = {
    name: '',
    email: '',
    message: ''
  };
  
  const handleSubmit = (values, { setSubmitting, resetForm }) => {
    console.log('Form values:', values);
    
    setTimeout(() => {
      alert(JSON.stringify(values, null, 2));
      setSubmitting(false);
      resetForm();
    }, 1000);
  };
  
  return (
    <div className="contact-form">
      <h1>Contact Us</h1>
      
      <Formik
        initialValues={initialValues}
        validationSchema={validationSchema}
        onSubmit={handleSubmit}
      >
        {({ isSubmitting, errors, touched }) => (
          <Form>
            <div className="form-group">
              <label htmlFor="name">Name</label>
              <Field
                id="name"
                name="name"
                type="text"
                className={touched.name && errors.name ? 'error' : ''}
              />
              <ErrorMessage name="name" component="div" className="error-message" />
            </div>
            
            <div className="form-group">
              <label htmlFor="email">Email</label>
              <Field
                id="email"
                name="email"
                type="email"
                className={touched.email && errors.email ? 'error' : ''}
              />
              <ErrorMessage name="email" component="div" className="error-message" />
            </div>
            
            <div className="form-group">
              <label htmlFor="message">Message</label>
              <Field
                as="textarea"
                id="message"
                name="message"
                rows="5"
                className={touched.message && errors.message ? 'error' : ''}
              />
              <ErrorMessage name="message" component="div" className="error-message" />
            </div>
            
            <button type="submit" disabled={isSubmitting}>
              {isSubmitting ? 'Sending...' : 'Send Message'}
            </button>
          </Form>
        )}
      </Formik>
    </div>
  );
}

export default ContactForm;
```

---

## Advanced Validation

### Conditional Validation

**Validate field based on another field:**

```jsx
const schema = yup.object().shape({
  hasCompany: yup.boolean(),
  
  companyName: yup.string().when('hasCompany', {
    is: true,
    then: (schema) => schema.required('Company name is required'),
    otherwise: (schema) => schema.notRequired()
  }),
  
  // Alternative syntax (v1.0+)
  companyName: yup.string().when('hasCompany', ([hasCompany], schema) => {
    return hasCompany ? schema.required('Required') : schema.notRequired();
  })
});
```

---

### Cross-field Validation

**Compare two fields:**

```jsx
const schema = yup.object().shape({
  password: yup
    .string()
    .min(8, 'Password must be at least 8 characters')
    .required('Password is required'),
  
  confirmPassword: yup
    .string()
    .oneOf([yup.ref('password'), null], 'Passwords must match')
    .required('Please confirm your password'),
  
  // Date range
  startDate: yup.date().required('Start date is required'),
  
  endDate: yup
    .date()
    .min(yup.ref('startDate'), 'End date must be after start date')
    .required('End date is required')
});
```

---

### Custom Validation

**Define custom validation logic:**

```jsx
const schema = yup.object().shape({
  username: yup
    .string()
    .required('Username is required')
    .test('is-valid-username', 'Username cannot be "admin"', (value) => {
      return value !== 'admin';
    }),
  
  password: yup
    .string()
    .required('Password is required')
    .test('password-strength', 'Password is too weak', (value) => {
      // Custom strength check
      return /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/.test(value);
    }),
  
  email: yup
    .string()
    .email()
    .test('email-domain', 'Only company emails allowed', (value) => {
      return value?.endsWith('@company.com');
    })
});
```

---

### Async Validation

**Check availability with API:**

```jsx
const schema = yup.object().shape({
  username: yup
    .string()
    .required('Username is required')
    .test('check-username', 'Username is already taken', async (value) => {
      if (!value) return true;
      
      const response = await fetch(`/api/check-username?username=${value}`);
      const data = await response.json();
      return data.available;
    }),
  
  email: yup
    .string()
    .email('Invalid email')
    .required('Email is required')
    .test('check-email', 'Email is already registered', async (value) => {
      if (!value) return true;
      
      const response = await fetch(`/api/check-email?email=${value}`);
      const data = await response.json();
      return data.available;
    })
});
```

---

### Nested Object Validation

```jsx
const schema = yup.object().shape({
  user: yup.object().shape({
    name: yup.string().required('Name is required'),
    email: yup.string().email().required('Email is required')
  }),
  
  address: yup.object().shape({
    street: yup.string().required('Street is required'),
    city: yup.string().required('City is required'),
    state: yup.string().required('State is required'),
    zipCode: yup
      .string()
      .matches(/^\d{5}$/, 'Invalid zip code')
      .required('Zip code is required')
  }),
  
  contacts: yup.array().of(
    yup.object().shape({
      name: yup.string().required('Contact name is required'),
      phone: yup.string().required('Phone number is required')
    })
  ).min(1, 'At least one contact is required')
});
```

---

### Array of Objects

```jsx
const schema = yup.object().shape({
  items: yup.array().of(
    yup.object().shape({
      name: yup.string().required('Item name is required'),
      quantity: yup
        .number()
        .positive('Quantity must be positive')
        .integer('Quantity must be an integer')
        .required('Quantity is required'),
      price: yup
        .number()
        .positive('Price must be positive')
        .required('Price is required')
    })
  ).min(1, 'At least one item is required')
});

// Usage with React Hook Form
function OrderForm() {
  const { register, control, handleSubmit, formState: { errors } } = useForm({
    resolver: yupResolver(schema),
    defaultValues: {
      items: [{ name: '', quantity: 1, price: 0 }]
    }
  });
  
  const { fields, append, remove } = useFieldArray({
    control,
    name: 'items'
  });
  
  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      {fields.map((field, index) => (
        <div key={field.id}>
          <input {...register(`items.${index}.name`)} placeholder="Name" />
          {errors.items?.[index]?.name && (
            <span>{errors.items[index].name.message}</span>
          )}
          
          <input
            type="number"
            {...register(`items.${index}.quantity`)}
            placeholder="Quantity"
          />
          {errors.items?.[index]?.quantity && (
            <span>{errors.items[index].quantity.message}</span>
          )}
          
          <input
            type="number"
            {...register(`items.${index}.price`)}
            placeholder="Price"
          />
          {errors.items?.[index]?.price && (
            <span>{errors.items[index].price.message}</span>
          )}
          
          <button type="button" onClick={() => remove(index)}>Remove</button>
        </div>
      ))}
      
      <button type="button" onClick={() => append({ name: '', quantity: 1, price: 0 })}>
        Add Item
      </button>
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Yup Methods

### String Methods

```jsx
yup.string()
  .required('Required')
  .min(3, 'Min 3 characters')
  .max(20, 'Max 20 characters')
  .matches(/regex/, 'Pattern error')
  .email('Invalid email')
  .url('Invalid URL')
  .trim()  // Trim whitespace
  .lowercase()  // Convert to lowercase
  .uppercase()  // Convert to uppercase
```

---

### Number Methods

```jsx
yup.number()
  .required('Required')
  .min(0, 'Min value is 0')
  .max(100, 'Max value is 100')
  .positive('Must be positive')
  .negative('Must be negative')
  .integer('Must be integer')
  .moreThan(10, 'Must be more than 10')
  .lessThan(100, 'Must be less than 100')
```

---

### Date Methods

```jsx
yup.date()
  .required('Required')
  .min(new Date(), 'Must be in future')
  .max(new Date(), 'Must be in past')
```

---

### Boolean Methods

```jsx
yup.boolean()
  .required('Required')
  .oneOf([true], 'Must be true')
```

---

### Array Methods

```jsx
yup.array()
  .required('Required')
  .min(1, 'At least 1 item')
  .max(5, 'Max 5 items')
  .of(yup.string())  // Array of strings
  .compact()  // Remove falsy values
```

---

### Mixed Methods

**Available on all types:**

```jsx
schema
  .required('Required')
  .nullable()  // Allow null
  .optional()  // Not required
  .default('default value')
  .transform((value, originalValue) => {
    // Transform value
    return value;
  })
  .test('test-name', 'Error message', (value) => {
    // Custom validation
    return true;
  })
  .when('otherField', {
    is: value => value,
    then: schema => schema.required(),
    otherwise: schema => schema.notRequired()
  })
```

---

## Manual Validation

**Validate data without forms:**

```jsx
import * as yup from 'yup';

const schema = yup.object().shape({
  email: yup.string().email().required(),
  age: yup.number().min(18).required()
});

// Validate
try {
  const validData = await schema.validate(
    { email: 'test@example.com', age: 25 },
    { abortEarly: false }  // Get all errors, not just first one
  );
  console.log('Valid:', validData);
} catch (error) {
  console.log('Errors:', error.errors);  // Array of error messages
  console.log('Inner errors:', error.inner);  // Detailed errors
}

// Validate synchronously
try {
  schema.validateSync({ email: 'test@example.com', age: 25 });
} catch (error) {
  console.log(error.message);
}

// Check if valid (no throw)
const isValid = await schema.isValid({ email: 'test@example.com', age: 25 });
console.log('Is valid:', isValid);  // true or false
```

---

## TypeScript Support

```typescript
import * as yup from 'yup';
import { InferType } from 'yup';

const userSchema = yup.object().shape({
  name: yup.string().required(),
  email: yup.string().email().required(),
  age: yup.number().min(18).required()
});

// Infer TypeScript type from schema
type User = InferType<typeof userSchema>;

// User = {
//   name: string;
//   email: string;
//   age: number;
// }

// Use with React Hook Form
const { register } = useForm<User>({
  resolver: yupResolver(userSchema)
});
```

---

## Reusable Schemas

**Create and compose schemas:**

```jsx
// Base schemas
const emailSchema = yup
  .string()
  .email('Invalid email')
  .required('Email is required');

const passwordSchema = yup
  .string()
  .min(8, 'Min 8 characters')
  .matches(/[A-Z]/, 'Must contain uppercase')
  .matches(/[a-z]/, 'Must contain lowercase')
  .matches(/\d/, 'Must contain number')
  .required('Password is required');

const nameSchema = yup
  .string()
  .min(2, 'Min 2 characters')
  .max(50, 'Max 50 characters')
  .required('Name is required');

// Compose schemas
const loginSchema = yup.object().shape({
  email: emailSchema,
  password: passwordSchema
});

const signupSchema = yup.object().shape({
  firstName: nameSchema,
  lastName: nameSchema,
  email: emailSchema,
  password: passwordSchema,
  confirmPassword: yup
    .string()
    .oneOf([yup.ref('password')], 'Passwords must match')
    .required('Confirm password')
});
```

---

## Best Practices

**1. Define schema outside component**
```jsx
// ✅ Good - defined once
const schema = yup.object().shape({ /* ... */ });

function MyForm() {
  const { register } = useForm({ resolver: yupResolver(schema) });
  // ...
}

// ❌ Bad - recreated on every render
function MyForm() {
  const schema = yup.object().shape({ /* ... */ });
  // ...
}
```

**2. Use descriptive error messages**
```jsx
// ✅ Clear
.required('Email address is required')
.email('Please enter a valid email address')

// ❌ Unclear
.required('Required')
.email('Invalid')
```

**3. Group related validations**
```jsx
// ✅ Reusable
const passwordSchema = yup.string()
  .min(8)
  .matches(/[A-Z]/)
  .matches(/[a-z]/)
  .matches(/\d/)
  .required();
```

**4. Use TypeScript inference**
```typescript
type FormData = InferType<typeof schema>;
```

**5. Handle async validation carefully**
```jsx
// Debounce async validation to avoid too many API calls
```

---

## Yup vs Zod

| Feature | Yup | Zod |
|---------|-----|-----|
| **TypeScript** | Good | Excellent (TS-first) |
| **Bundle Size** | ~14KB | ~8KB |
| **API Style** | Chainable | Chainable |
| **Async** | Yes | Yes |
| **Parsing** | Casting | Type-safe parsing |
| **Popularity** | Very popular | Growing |
| **Learning Curve** | Easy | Easy |
| **Best For** | React forms | TypeScript projects |

---

**Interview Tips:**
- Yup = **JavaScript schema validation** library
- **Declarative** validation (schema-based)
- Created by **Jason Quense**
- Works with **Formik** and **React Hook Form**
- **Schema** = object shape definition
- **yup.object().shape({})** defines object schema
- **String validation**: required, min, max, matches, email, url
- **Number validation**: min, max, positive, integer
- **Date validation**: min, max
- **Boolean validation**: oneOf
- **Array validation**: of, min, max
- **Cross-field validation**: yup.ref() to reference other fields
- **Conditional validation**: when() for conditional rules
- **Custom validation**: test() method
- **Async validation**: async test() method
- **Nested objects**: nested shape definitions
- **Error messages**: customizable for each rule
- **TypeScript**: InferType for type inference
- **@hookform/resolvers/yup** for React Hook Form
- **validationSchema** prop for Formik
- **validate()** to validate data manually
- **isValid()** to check validity without throwing
- **abortEarly: false** to get all errors
- **Transform**: modify value before validation
- **Default values**: default() method
- **Nullable**: nullable() to allow null
- **Optional**: notRequired() or optional()
- Smaller than **Formik** (~14KB vs ~15KB)
- **Reusable schemas** for consistency
- **Composable** - build complex from simple
- Define schema **outside component** (performance)
- Most popular **schema validator** for React
- Alternative: **Zod** (TypeScript-first)

</details>

---

### 75. How do you handle file uploads in React?

<details>
<summary>View Answer</summary>

**File Uploads in React**

File uploads in React involve handling file input elements, reading file data, validating files, and sending them to a server.

---

## Basic File Upload

### Single File Upload

```jsx
import React, { useState } from 'react';

function FileUpload() {
  const [file, setFile] = useState(null);
  const [uploading, setUploading] = useState(false);
  const [uploadStatus, setUploadStatus] = useState('');
  
  const handleFileChange = (e) => {
    const selectedFile = e.target.files[0];
    setFile(selectedFile);
    console.log('Selected file:', selectedFile);
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!file) {
      alert('Please select a file');
      return;
    }
    
    setUploading(true);
    setUploadStatus('');
    
    try {
      // Create FormData
      const formData = new FormData();
      formData.append('file', file);
      
      // Upload to server
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData
      });
      
      if (response.ok) {
        setUploadStatus('File uploaded successfully!');
        setFile(null);
      } else {
        setUploadStatus('Upload failed');
      }
    } catch (error) {
      console.error('Upload error:', error);
      setUploadStatus('Upload failed');
    } finally {
      setUploading(false);
    }
  };
  
  return (
    <div className="file-upload">
      <h2>Upload File</h2>
      
      <form onSubmit={handleSubmit}>
        <input
          type="file"
          onChange={handleFileChange}
          disabled={uploading}
        />
        
        {file && (
          <div className="file-info">
            <p>File: {file.name}</p>
            <p>Size: {(file.size / 1024).toFixed(2)} KB</p>
            <p>Type: {file.type}</p>
          </div>
        )}
        
        <button type="submit" disabled={!file || uploading}>
          {uploading ? 'Uploading...' : 'Upload'}
        </button>
        
        {uploadStatus && <p>{uploadStatus}</p>}
      </form>
    </div>
  );
}

export default FileUpload;
```

---

### Multiple File Upload

```jsx
import React, { useState } from 'react';

function MultipleFileUpload() {
  const [files, setFiles] = useState([]);
  const [uploading, setUploading] = useState(false);
  
  const handleFileChange = (e) => {
    const selectedFiles = Array.from(e.target.files);
    setFiles(selectedFiles);
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (files.length === 0) {
      alert('Please select files');
      return;
    }
    
    setUploading(true);
    
    try {
      const formData = new FormData();
      
      // Append all files
      files.forEach((file, index) => {
        formData.append(`file${index}`, file);
        // or formData.append('files', file); // Same name for all files
      });
      
      const response = await fetch('/api/upload-multiple', {
        method: 'POST',
        body: formData
      });
      
      if (response.ok) {
        alert('Files uploaded successfully!');
        setFiles([]);
      }
    } catch (error) {
      console.error('Upload error:', error);
    } finally {
      setUploading(false);
    }
  };
  
  return (
    <div className="file-upload">
      <h2>Upload Multiple Files</h2>
      
      <form onSubmit={handleSubmit}>
        <input
          type="file"
          multiple
          onChange={handleFileChange}
          disabled={uploading}
        />
        
        {files.length > 0 && (
          <div className="files-list">
            <h3>Selected Files ({files.length}):</h3>
            <ul>
              {files.map((file, index) => (
                <li key={index}>
                  {file.name} - {(file.size / 1024).toFixed(2)} KB
                </li>
              ))}
            </ul>
          </div>
        )}
        
        <button type="submit" disabled={files.length === 0 || uploading}>
          {uploading ? 'Uploading...' : 'Upload All'}
        </button>
      </form>
    </div>
  );
}

export default MultipleFileUpload;
```

---

## File Validation

### Validate File Type and Size

```jsx
import React, { useState } from 'react';

function FileUploadWithValidation() {
  const [file, setFile] = useState(null);
  const [error, setError] = useState('');
  const [preview, setPreview] = useState(null);
  
  const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
  const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
  
  const validateFile = (file) => {
    // Check file type
    if (!ALLOWED_TYPES.includes(file.type)) {
      return 'Only JPEG, PNG, GIF, and WebP images are allowed';
    }
    
    // Check file size
    if (file.size > MAX_FILE_SIZE) {
      return 'File size must be less than 5MB';
    }
    
    return null;
  };
  
  const handleFileChange = (e) => {
    const selectedFile = e.target.files[0];
    
    if (!selectedFile) {
      setFile(null);
      setError('');
      setPreview(null);
      return;
    }
    
    const validationError = validateFile(selectedFile);
    
    if (validationError) {
      setError(validationError);
      setFile(null);
      setPreview(null);
      return;
    }
    
    setFile(selectedFile);
    setError('');
    
    // Create preview for images
    const reader = new FileReader();
    reader.onloadend = () => {
      setPreview(reader.result);
    };
    reader.readAsDataURL(selectedFile);
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!file) return;
    
    const formData = new FormData();
    formData.append('file', file);
    
    try {
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData
      });
      
      if (response.ok) {
        alert('File uploaded successfully!');
        setFile(null);
        setPreview(null);
      }
    } catch (error) {
      console.error('Upload error:', error);
    }
  };
  
  return (
    <div className="file-upload">
      <h2>Upload Image</h2>
      
      <form onSubmit={handleSubmit}>
        <div className="file-input-wrapper">
          <input
            type="file"
            accept="image/*"
            onChange={handleFileChange}
          />
        </div>
        
        {error && <p className="error">{error}</p>}
        
        {preview && (
          <div className="preview">
            <h3>Preview:</h3>
            <img src={preview} alt="Preview" style={{ maxWidth: '300px' }} />
            <p>{file.name} ({(file.size / 1024).toFixed(2)} KB)</p>
          </div>
        )}
        
        <button type="submit" disabled={!file}>
          Upload
        </button>
      </form>
    </div>
  );
}

export default FileUploadWithValidation;
```

---

## Drag and Drop Upload

```jsx
import React, { useState, useRef } from 'react';

function DragDropUpload() {
  const [files, setFiles] = useState([]);
  const [isDragging, setIsDragging] = useState(false);
  const fileInputRef = useRef(null);
  
  const handleDragEnter = (e) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDragging(true);
  };
  
  const handleDragLeave = (e) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDragging(false);
  };
  
  const handleDragOver = (e) => {
    e.preventDefault();
    e.stopPropagation();
  };
  
  const handleDrop = (e) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDragging(false);
    
    const droppedFiles = Array.from(e.dataTransfer.files);
    setFiles(droppedFiles);
  };
  
  const handleFileSelect = (e) => {
    const selectedFiles = Array.from(e.target.files);
    setFiles(selectedFiles);
  };
  
  const handleUpload = async () => {
    if (files.length === 0) return;
    
    const formData = new FormData();
    files.forEach((file) => {
      formData.append('files', file);
    });
    
    try {
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData
      });
      
      if (response.ok) {
        alert('Files uploaded successfully!');
        setFiles([]);
      }
    } catch (error) {
      console.error('Upload error:', error);
    }
  };
  
  const removeFile = (index) => {
    setFiles(files.filter((_, i) => i !== index));
  };
  
  return (
    <div className="drag-drop-upload">
      <h2>Drag & Drop File Upload</h2>
      
      <div
        className={`drop-zone ${isDragging ? 'dragging' : ''}`}
        onDragEnter={handleDragEnter}
        onDragLeave={handleDragLeave}
        onDragOver={handleDragOver}
        onDrop={handleDrop}
        onClick={() => fileInputRef.current.click()}
      >
        <input
          ref={fileInputRef}
          type="file"
          multiple
          onChange={handleFileSelect}
          style={{ display: 'none' }}
        />
        
        <div className="drop-zone-content">
          <p>Drag & drop files here</p>
          <p>or click to browse</p>
        </div>
      </div>
      
      {files.length > 0 && (
        <div className="files-list">
          <h3>Selected Files:</h3>
          <ul>
            {files.map((file, index) => (
              <li key={index}>
                <span>{file.name}</span>
                <span>{(file.size / 1024).toFixed(2)} KB</span>
                <button onClick={() => removeFile(index)}>Remove</button>
              </li>
            ))}
          </ul>
          
          <button onClick={handleUpload}>Upload All</button>
        </div>
      )}
    </div>
  );
}

export default DragDropUpload;
```

**CSS for Drag & Drop:**
```css
.drop-zone {
  border: 2px dashed #ccc;
  border-radius: 8px;
  padding: 40px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
}

.drop-zone:hover {
  border-color: #4CAF50;
  background-color: #f0f8f0;
}

.drop-zone.dragging {
  border-color: #4CAF50;
  background-color: #e8f5e9;
}

.drop-zone-content p {
  margin: 10px 0;
  color: #666;
}
```

---

## Upload Progress

```jsx
import React, { useState } from 'react';
import axios from 'axios';

function FileUploadWithProgress() {
  const [file, setFile] = useState(null);
  const [uploadProgress, setUploadProgress] = useState(0);
  const [uploading, setUploading] = useState(false);
  
  const handleFileChange = (e) => {
    setFile(e.target.files[0]);
    setUploadProgress(0);
  };
  
  const handleUpload = async () => {
    if (!file) return;
    
    setUploading(true);
    const formData = new FormData();
    formData.append('file', file);
    
    try {
      await axios.post('/api/upload', formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        },
        onUploadProgress: (progressEvent) => {
          const percentCompleted = Math.round(
            (progressEvent.loaded * 100) / progressEvent.total
          );
          setUploadProgress(percentCompleted);
        }
      });
      
      alert('Upload complete!');
      setFile(null);
      setUploadProgress(0);
    } catch (error) {
      console.error('Upload error:', error);
    } finally {
      setUploading(false);
    }
  };
  
  return (
    <div className="file-upload-progress">
      <h2>Upload with Progress</h2>
      
      <input type="file" onChange={handleFileChange} disabled={uploading} />
      
      {file && (
        <div>
          <p>{file.name}</p>
          
          <button onClick={handleUpload} disabled={uploading}>
            {uploading ? 'Uploading...' : 'Upload'}
          </button>
          
          {uploading && (
            <div className="progress-bar">
              <div
                className="progress-fill"
                style={{ width: `${uploadProgress}%` }}
              >
                {uploadProgress}%
              </div>
            </div>
          )}
        </div>
      )}
    </div>
  );
}

export default FileUploadWithProgress;
```

**CSS for Progress Bar:**
```css
.progress-bar {
  width: 100%;
  height: 30px;
  background-color: #f0f0f0;
  border-radius: 15px;
  overflow: hidden;
  margin-top: 10px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #4CAF50, #45a049);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: bold;
  transition: width 0.3s ease;
}
```

---

## Image Preview

```jsx
import React, { useState } from 'react';

function ImageUploadWithPreview() {
  const [images, setImages] = useState([]);
  
  const handleFileChange = (e) => {
    const files = Array.from(e.target.files);
    
    // Create preview URLs
    const imagePromises = files.map((file) => {
      return new Promise((resolve) => {
        const reader = new FileReader();
        reader.onload = (e) => {
          resolve({
            file,
            preview: e.target.result,
            name: file.name,
            size: file.size
          });
        };
        reader.readAsDataURL(file);
      });
    });
    
    Promise.all(imagePromises).then((images) => {
      setImages(images);
    });
  };
  
  const removeImage = (index) => {
    setImages(images.filter((_, i) => i !== index));
  };
  
  const handleUpload = async () => {
    const formData = new FormData();
    images.forEach((image) => {
      formData.append('images', image.file);
    });
    
    try {
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData
      });
      
      if (response.ok) {
        alert('Images uploaded!');
        setImages([]);
      }
    } catch (error) {
      console.error('Upload error:', error);
    }
  };
  
  return (
    <div className="image-upload">
      <h2>Upload Images</h2>
      
      <input
        type="file"
        accept="image/*"
        multiple
        onChange={handleFileChange}
      />
      
      {images.length > 0 && (
        <div className="image-grid">
          {images.map((image, index) => (
            <div key={index} className="image-item">
              <img src={image.preview} alt={image.name} />
              <div className="image-info">
                <p>{image.name}</p>
                <p>{(image.size / 1024).toFixed(2)} KB</p>
              </div>
              <button onClick={() => removeImage(index)}>Remove</button>
            </div>
          ))}
        </div>
      )}
      
      {images.length > 0 && (
        <button onClick={handleUpload}>Upload {images.length} Images</button>
      )}
    </div>
  );
}

export default ImageUploadWithPreview;
```

---

## With React Hook Form

```jsx
import React from 'react';
import { useForm } from 'react-hook-form';

function FileUploadForm() {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors }
  } = useForm();
  
  const fileList = watch('file');
  
  const onSubmit = async (data) => {
    const formData = new FormData();
    
    // Append file
    if (data.file[0]) {
      formData.append('file', data.file[0]);
    }
    
    // Append other fields
    formData.append('title', data.title);
    formData.append('description', data.description);
    
    try {
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData
      });
      
      if (response.ok) {
        alert('File uploaded successfully!');
      }
    } catch (error) {
      console.error('Upload error:', error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Title</label>
        <input
          {...register('title', { required: 'Title is required' })}
        />
        {errors.title && <span>{errors.title.message}</span>}
      </div>
      
      <div>
        <label>Description</label>
        <textarea
          {...register('description', { required: 'Description is required' })}
        />
        {errors.description && <span>{errors.description.message}</span>}
      </div>
      
      <div>
        <label>File</label>
        <input
          type="file"
          {...register('file', {
            required: 'File is required',
            validate: {
              fileSize: (files) => {
                if (!files[0]) return true;
                return files[0].size <= 5000000 || 'File size must be less than 5MB';
              },
              fileType: (files) => {
                if (!files[0]) return true;
                const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
                return allowedTypes.includes(files[0].type) || 'Only JPEG, PNG, GIF allowed';
              }
            }
          })}
        />
        {errors.file && <span>{errors.file.message}</span>}
      </div>
      
      {fileList && fileList[0] && (
        <div>
          <p>Selected: {fileList[0].name}</p>
          <p>Size: {(fileList[0].size / 1024).toFixed(2)} KB</p>
        </div>
      )}
      
      <button type="submit">Upload</button>
    </form>
  );
}

export default FileUploadForm;
```

---

## With Formik

```jsx
import React from 'react';
import { Formik, Form, Field } from 'formik';
import * as Yup from 'yup';

const validationSchema = Yup.object().shape({
  title: Yup.string().required('Title is required'),
  file: Yup.mixed()
    .required('File is required')
    .test('fileSize', 'File size is too large (max 5MB)', (value) => {
      return value && value.size <= 5000000;
    })
    .test('fileType', 'Unsupported file format', (value) => {
      return value && ['image/jpeg', 'image/png', 'image/gif'].includes(value.type);
    })
});

function FormikFileUpload() {
  return (
    <Formik
      initialValues={{
        title: '',
        file: null
      }}
      validationSchema={validationSchema}
      onSubmit={async (values) => {
        const formData = new FormData();
        formData.append('title', values.title);
        formData.append('file', values.file);
        
        try {
          const response = await fetch('/api/upload', {
            method: 'POST',
            body: formData
          });
          
          if (response.ok) {
            alert('File uploaded!');
          }
        } catch (error) {
          console.error('Upload error:', error);
        }
      }}
    >
      {({ setFieldValue, values, errors, touched }) => (
        <Form>
          <div>
            <label>Title</label>
            <Field name="title" />
            {errors.title && touched.title && <div>{errors.title}</div>}
          </div>
          
          <div>
            <label>File</label>
            <input
              type="file"
              name="file"
              onChange={(event) => {
                setFieldValue('file', event.currentTarget.files[0]);
              }}
            />
            {errors.file && touched.file && <div>{errors.file}</div>}
          </div>
          
          {values.file && (
            <div>
              <p>Selected: {values.file.name}</p>
            </div>
          )}
          
          <button type="submit">Upload</button>
        </Form>
      )}
    </Formik>
  );
}

export default FormikFileUpload;
```

---

## Base64 Encoding

**Convert file to Base64:**

```jsx
import React, { useState } from 'react';

function Base64Upload() {
  const [base64, setBase64] = useState('');
  const [fileName, setFileName] = useState('');
  
  const handleFileChange = (e) => {
    const file = e.target.files[0];
    if (!file) return;
    
    setFileName(file.name);
    
    const reader = new FileReader();
    reader.onloadend = () => {
      setBase64(reader.result);
    };
    reader.readAsDataURL(file);
  };
  
  const handleSubmit = async () => {
    try {
      const response = await fetch('/api/upload-base64', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          fileName,
          fileData: base64
        })
      });
      
      if (response.ok) {
        alert('File uploaded!');
      }
    } catch (error) {
      console.error('Upload error:', error);
    }
  };
  
  return (
    <div>
      <input type="file" onChange={handleFileChange} />
      
      {base64 && (
        <div>
          <p>File: {fileName}</p>
          <button onClick={handleSubmit}>Upload</button>
        </div>
      )}
    </div>
  );
}

export default Base64Upload;
```

---

## Chunked Upload (Large Files)

**Upload large files in chunks:**

```jsx
import React, { useState } from 'react';

function ChunkedUpload() {
  const [file, setFile] = useState(null);
  const [progress, setProgress] = useState(0);
  const [uploading, setUploading] = useState(false);
  
  const CHUNK_SIZE = 1024 * 1024; // 1MB chunks
  
  const uploadChunk = async (chunk, chunkIndex, totalChunks, fileName) => {
    const formData = new FormData();
    formData.append('chunk', chunk);
    formData.append('chunkIndex', chunkIndex);
    formData.append('totalChunks', totalChunks);
    formData.append('fileName', fileName);
    
    const response = await fetch('/api/upload-chunk', {
      method: 'POST',
      body: formData
    });
    
    return response.ok;
  };
  
  const handleUpload = async () => {
    if (!file) return;
    
    setUploading(true);
    setProgress(0);
    
    const totalChunks = Math.ceil(file.size / CHUNK_SIZE);
    
    try {
      for (let chunkIndex = 0; chunkIndex < totalChunks; chunkIndex++) {
        const start = chunkIndex * CHUNK_SIZE;
        const end = Math.min(start + CHUNK_SIZE, file.size);
        const chunk = file.slice(start, end);
        
        const success = await uploadChunk(chunk, chunkIndex, totalChunks, file.name);
        
        if (!success) {
          throw new Error('Chunk upload failed');
        }
        
        setProgress(Math.round(((chunkIndex + 1) / totalChunks) * 100));
      }
      
      alert('File uploaded successfully!');
    } catch (error) {
      console.error('Upload error:', error);
      alert('Upload failed');
    } finally {
      setUploading(false);
    }
  };
  
  return (
    <div>
      <h2>Chunked Upload (Large Files)</h2>
      
      <input
        type="file"
        onChange={(e) => setFile(e.target.files[0])}
        disabled={uploading}
      />
      
      {file && (
        <div>
          <p>{file.name} ({(file.size / (1024 * 1024)).toFixed(2)} MB)</p>
          
          <button onClick={handleUpload} disabled={uploading}>
            {uploading ? 'Uploading...' : 'Upload'}
          </button>
          
          {uploading && (
            <div className="progress-bar">
              <div className="progress-fill" style={{ width: `${progress}%` }}>
                {progress}%
              </div>
            </div>
          )}
        </div>
      )}
    </div>
  );
}

export default ChunkedUpload;
```

---

## Complete Example: Profile Picture Upload

```jsx
import React, { useState } from 'react';

function ProfilePictureUpload() {
  const [selectedFile, setSelectedFile] = useState(null);
  const [preview, setPreview] = useState(null);
  const [uploading, setUploading] = useState(false);
  const [error, setError] = useState('');
  
  const MAX_SIZE = 2 * 1024 * 1024; // 2MB
  const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/gif'];
  
  const handleFileSelect = (e) => {
    const file = e.target.files[0];
    setError('');
    
    if (!file) {
      setSelectedFile(null);
      setPreview(null);
      return;
    }
    
    // Validate file type
    if (!ALLOWED_TYPES.includes(file.type)) {
      setError('Only JPEG, PNG, and GIF images are allowed');
      return;
    }
    
    // Validate file size
    if (file.size > MAX_SIZE) {
      setError('Image must be less than 2MB');
      return;
    }
    
    setSelectedFile(file);
    
    // Create preview
    const reader = new FileReader();
    reader.onloadend = () => {
      setPreview(reader.result);
    };
    reader.readAsDataURL(file);
  };
  
  const handleUpload = async () => {
    if (!selectedFile) return;
    
    setUploading(true);
    setError('');
    
    const formData = new FormData();
    formData.append('profilePicture', selectedFile);
    
    try {
      const response = await fetch('/api/user/profile-picture', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        },
        body: formData
      });
      
      if (response.ok) {
        const data = await response.json();
        alert('Profile picture updated!');
        console.log('Image URL:', data.imageUrl);
      } else {
        setError('Upload failed. Please try again.');
      }
    } catch (error) {
      console.error('Upload error:', error);
      setError('Upload failed. Please try again.');
    } finally {
      setUploading(false);
    }
  };
  
  const handleRemove = () => {
    setSelectedFile(null);
    setPreview(null);
    setError('');
  };
  
  return (
    <div className="profile-picture-upload">
      <h2>Upload Profile Picture</h2>
      
      <div className="upload-area">
        {preview ? (
          <div className="preview-container">
            <img
              src={preview}
              alt="Preview"
              className="preview-image"
            />
            <div className="preview-actions">
              <button onClick={handleRemove} className="btn-remove">
                Remove
              </button>
              <button
                onClick={handleUpload}
                disabled={uploading}
                className="btn-upload"
              >
                {uploading ? 'Uploading...' : 'Upload'}
              </button>
            </div>
          </div>
        ) : (
          <label className="upload-label">
            <input
              type="file"
              accept="image/*"
              onChange={handleFileSelect}
              style={{ display: 'none' }}
            />
            <div className="upload-placeholder">
              <svg
                width="64"
                height="64"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
              >
                <path d="M12 5v14M5 12h14" strokeWidth="2" strokeLinecap="round" />
              </svg>
              <p>Click to upload profile picture</p>
              <p className="upload-hint">JPEG, PNG, or GIF (max 2MB)</p>
            </div>
          </label>
        )}
      </div>
      
      {error && <p className="error-message">{error}</p>}
    </div>
  );
}

export default ProfilePictureUpload;
```

**CSS:**
```css
.profile-picture-upload {
  max-width: 400px;
  margin: 0 auto;
}

.upload-area {
  border: 2px dashed #ddd;
  border-radius: 8px;
  padding: 20px;
  text-align: center;
}

.upload-label {
  cursor: pointer;
  display: block;
}

.upload-placeholder {
  color: #666;
}

.upload-placeholder svg {
  color: #999;
  margin-bottom: 10px;
}

.upload-hint {
  font-size: 12px;
  color: #999;
  margin-top: 5px;
}

.preview-container {
  text-align: center;
}

.preview-image {
  max-width: 200px;
  max-height: 200px;
  border-radius: 50%;
  object-fit: cover;
  margin-bottom: 15px;
}

.preview-actions {
  display: flex;
  gap: 10px;
  justify-content: center;
}

.btn-remove,
.btn-upload {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 500;
}

.btn-remove {
  background-color: #f44336;
  color: white;
}

.btn-upload {
  background-color: #4CAF50;
  color: white;
}

.btn-upload:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.error-message {
  color: #f44336;
  margin-top: 10px;
  font-size: 14px;
}
```

---

## Best Practices

**1. Validate files on the client side**
```jsx
// Check file type and size before upload
if (file.size > MAX_SIZE) {
  // Show error
}
```

**2. Always validate on the server too**
```javascript
// Server-side validation is essential for security
```

**3. Show upload progress for large files**
```jsx
// Use axios with onUploadProgress
```

**4. Handle errors gracefully**
```jsx
try {
  await uploadFile();
} catch (error) {
  setError('Upload failed. Please try again.');
}
```

**5. Limit file types**
```jsx
<input type="file" accept="image/*" />
// or accept=".pdf,.doc,.docx"
```

**6. Clean up preview URLs**
```jsx
useEffect(() => {
  return () => {
    if (preview) {
      URL.revokeObjectURL(preview);
    }
  };
}, [preview]);
```

**7. Use FormData for file uploads**
```jsx
const formData = new FormData();
formData.append('file', file);
```

---

**Interview Tips:**
- File uploads use **`<input type="file">`** element
- **e.target.files** returns FileList (array-like)
- **e.target.files[0]** for single file
- **Array.from(e.target.files)** for multiple files
- **FormData** API to send files to server
- **formData.append('key', file)** to add file
- **FileReader** API to read file contents
- **reader.readAsDataURL()** for preview (base64)
- **reader.readAsText()** for text files
- **reader.readAsArrayBuffer()** for binary
- **multiple** attribute for multiple files
- **accept** attribute to limit file types
- Validate **file.type** and **file.size**
- **file.name** = filename
- **file.size** = size in bytes
- **file.type** = MIME type
- **Drag and Drop** with onDrop event
- **e.dataTransfer.files** for dropped files
- **Upload progress** with axios onUploadProgress
- **Chunked upload** for large files
- **Preview images** with FileReader
- **Base64** encoding for API upload
- Always validate **client + server side**
- Use **refs** for custom file input
- **URL.createObjectURL()** for preview (faster)
- **URL.revokeObjectURL()** to clean up
- **React Hook Form**: register file input
- **Formik**: use setFieldValue for file
- **Common validations**: type, size, dimensions
- **Security**: validate file type on server
- **MIME type** can be spoofed (check server-side)
- **Progress bar** improves UX for large files
- **Async upload** with fetch or axios
- **Error handling** essential
- **Loading states** while uploading

</details>

---

### 76. What is Zod and how do you use it with forms?

<details>
<summary>View Answer</summary>

**Zod**

Zod is a **TypeScript-first schema validation library** with static type inference. It's similar to Yup but designed specifically for TypeScript with better type safety.

**Official website:** https://zod.dev/

**Created by:** Colin McDonnell

---

## Why Use Zod?

**Key Benefits:**
1. **TypeScript-first** - Built for TypeScript
2. **Type inference** - Infer types from schemas
3. **Zero dependencies** - Lightweight
4. **Composable** - Build complex schemas
5. **Runtime validation** - Validate data at runtime
6. **Better error messages** - Detailed validation errors
7. **Tree-shakeable** - Only bundle what you use

---

## Yup vs Zod

| Feature | Yup | Zod |
|---------|-----|-----|
| **TypeScript** | Good | Excellent (TS-first) |
| **Type Inference** | Manual | Automatic |
| **Bundle Size** | ~14KB | ~8KB |
| **Dependencies** | Yes | Zero |
| **API Style** | Chainable | Chainable |
| **Learning Curve** | Easy | Easy |
| **Ecosystem** | Mature | Growing |
| **Best For** | JavaScript + React | TypeScript projects |

---

## Installation

```bash
npm install zod
```

**With React Hook Form:**
```bash
npm install zod @hookform/resolvers
```

---

## Basic Usage

### String Validation

```typescript
import { z } from 'zod';

// Define schema
const userSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  username: z.string().min(3).max(20),
  url: z.string().url(),
  bio: z.string().optional()
});

// Validate data
const result = userSchema.safeParse({
  name: 'John Doe',
  email: 'john@example.com',
  username: 'johndoe',
  url: 'https://example.com'
});

if (result.success) {
  console.log('Valid data:', result.data);
} else {
  console.log('Errors:', result.error);
}
```

---

### Number Validation

```typescript
const schema = z.object({
  age: z.number().min(18).max(120),
  price: z.number().positive(),
  quantity: z.number().int().positive(),
  rating: z.number().min(1).max(5)
});
```

---

### Boolean Validation

```typescript
const schema = z.object({
  terms: z.boolean().refine(val => val === true, {
    message: 'You must accept the terms'
  }),
  newsletter: z.boolean().optional()
});
```

---

### Date Validation

```typescript
const schema = z.object({
  birthDate: z.date().max(new Date(), 'Birth date cannot be in the future'),
  appointmentDate: z.date().min(new Date(), 'Appointment must be in the future')
});
```

---

### Array Validation

```typescript
const schema = z.object({
  tags: z.array(z.string()).min(1).max(5),
  hobbies: z.array(z.string().min(2)),
  scores: z.array(z.number().min(0).max(100))
});
```

---

### Object Validation

```typescript
const schema = z.object({
  user: z.object({
    name: z.string(),
    email: z.string().email()
  }),
  address: z.object({
    street: z.string(),
    city: z.string(),
    zipCode: z.string().regex(/^\d{5}$/)
  })
});
```

---

## Type Inference

**Automatically infer TypeScript types:**

```typescript
import { z } from 'zod';

const userSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  age: z.number().min(18)
});

// Infer type from schema
type User = z.infer<typeof userSchema>;

// User = {
//   name: string;
//   email: string;
//   age: number;
// }

// Use the type
const createUser = (user: User) => {
  console.log(user);
};
```

---

## With React Hook Form

```typescript
import React from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Define schema
const signupSchema = z.object({
  firstName: z.string().min(2, 'First name must be at least 2 characters'),
  lastName: z.string().min(2, 'Last name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[a-z]/, 'Must contain lowercase letter')
    .regex(/[A-Z]/, 'Must contain uppercase letter')
    .regex(/\d/, 'Must contain number'),
  confirmPassword: z.string(),
  age: z.number().min(18, 'Must be 18 or older'),
  terms: z.boolean().refine(val => val === true, {
    message: 'You must accept the terms'
  })
}).refine(data => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword']
});

// Infer type
type SignupFormData = z.infer<typeof signupSchema>;

function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting }
  } = useForm<SignupFormData>({
    resolver: zodResolver(signupSchema)
  });
  
  const onSubmit = async (data: SignupFormData) => {
    console.log('Form data:', data);
    // API call
  };
  
  return (
    <div className="signup-form">
      <h1>Sign Up</h1>
      
      <form onSubmit={handleSubmit(onSubmit)}>
        {/* First Name */}
        <div className="form-group">
          <label htmlFor="firstName">First Name</label>
          <input
            id="firstName"
            {...register('firstName')}
            className={errors.firstName ? 'error' : ''}
          />
          {errors.firstName && (
            <span className="error-message">{errors.firstName.message}</span>
          )}
        </div>
        
        {/* Last Name */}
        <div className="form-group">
          <label htmlFor="lastName">Last Name</label>
          <input
            id="lastName"
            {...register('lastName')}
            className={errors.lastName ? 'error' : ''}
          />
          {errors.lastName && (
            <span className="error-message">{errors.lastName.message}</span>
          )}
        </div>
        
        {/* Email */}
        <div className="form-group">
          <label htmlFor="email">Email</label>
          <input
            id="email"
            type="email"
            {...register('email')}
            className={errors.email ? 'error' : ''}
          />
          {errors.email && (
            <span className="error-message">{errors.email.message}</span>
          )}
        </div>
        
        {/* Password */}
        <div className="form-group">
          <label htmlFor="password">Password</label>
          <input
            id="password"
            type="password"
            {...register('password')}
            className={errors.password ? 'error' : ''}
          />
          {errors.password && (
            <span className="error-message">{errors.password.message}</span>
          )}
        </div>
        
        {/* Confirm Password */}
        <div className="form-group">
          <label htmlFor="confirmPassword">Confirm Password</label>
          <input
            id="confirmPassword"
            type="password"
            {...register('confirmPassword')}
            className={errors.confirmPassword ? 'error' : ''}
          />
          {errors.confirmPassword && (
            <span className="error-message">{errors.confirmPassword.message}</span>
          )}
        </div>
        
        {/* Age */}
        <div className="form-group">
          <label htmlFor="age">Age</label>
          <input
            id="age"
            type="number"
            {...register('age', { valueAsNumber: true })}
            className={errors.age ? 'error' : ''}
          />
          {errors.age && (
            <span className="error-message">{errors.age.message}</span>
          )}
        </div>
        
        {/* Terms */}
        <div className="form-group checkbox">
          <label>
            <input type="checkbox" {...register('terms')} />
            I accept the terms and conditions
          </label>
          {errors.terms && (
            <span className="error-message">{errors.terms.message}</span>
          )}
        </div>
        
        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Signing Up...' : 'Sign Up'}
        </button>
      </form>
    </div>
  );
}

export default SignupForm;
```

---

## Advanced Validation

### Custom Validation (refine)

```typescript
const schema = z.object({
  username: z.string().refine(
    val => val !== 'admin',
    { message: 'Username cannot be "admin"' }
  ),
  
  password: z.string().refine(
    val => /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/.test(val),
    { message: 'Password must contain uppercase, lowercase, number, and special character' }
  ),
  
  email: z.string().email().refine(
    val => val.endsWith('@company.com'),
    { message: 'Only company emails are allowed' }
  )
});
```

---

### Cross-field Validation

```typescript
const schema = z.object({
  password: z.string().min(8),
  confirmPassword: z.string()
}).refine(data => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword']  // Show error on confirmPassword field
});

// Date range validation
const dateRangeSchema = z.object({
  startDate: z.date(),
  endDate: z.date()
}).refine(data => data.endDate > data.startDate, {
  message: 'End date must be after start date',
  path: ['endDate']
});
```

---

### Async Validation

```typescript
const schema = z.object({
  username: z.string().refine(
    async (val) => {
      const response = await fetch(`/api/check-username?username=${val}`);
      const data = await response.json();
      return data.available;
    },
    { message: 'Username is already taken' }
  ),
  
  email: z.string().email().refine(
    async (val) => {
      const response = await fetch(`/api/check-email?email=${val}`);
      const data = await response.json();
      return data.available;
    },
    { message: 'Email is already registered' }
  )
});
```

---

### Optional and Nullable

```typescript
const schema = z.object({
  name: z.string(),
  middleName: z.string().optional(),  // Can be undefined
  nickname: z.string().nullable(),    // Can be null
  bio: z.string().optional().nullable()  // Can be undefined or null
});

// With default values
const schema2 = z.object({
  theme: z.string().default('light'),
  notifications: z.boolean().default(true)
});
```

---

### Transformations

```typescript
const schema = z.object({
  email: z.string().email().toLowerCase(),  // Transform to lowercase
  name: z.string().trim(),  // Trim whitespace
  age: z.string().transform(val => parseInt(val, 10)),  // String to number
  tags: z.string().transform(val => val.split(',').map(s => s.trim()))  // CSV to array
});
```

---

### Union Types

```typescript
// Either string or number
const schema = z.object({
  id: z.union([z.string(), z.number()])
});

// Discriminated union
const petSchema = z.discriminatedUnion('type', [
  z.object({ type: z.literal('dog'), breed: z.string() }),
  z.object({ type: z.literal('cat'), color: z.string() })
]);
```

---

### Enum Validation

```typescript
const schema = z.object({
  role: z.enum(['admin', 'user', 'moderator']),
  status: z.enum(['active', 'inactive', 'pending'])
});

// Native enum
enum Role {
  Admin = 'admin',
  User = 'user',
  Moderator = 'moderator'
}

const schema2 = z.object({
  role: z.nativeEnum(Role)
});
```

---

### Array of Objects

```typescript
const itemSchema = z.object({
  name: z.string().min(1),
  quantity: z.number().int().positive(),
  price: z.number().positive()
});

const orderSchema = z.object({
  items: z.array(itemSchema).min(1, 'At least one item is required')
});

// Infer type
type Order = z.infer<typeof orderSchema>;

// Order = {
//   items: Array<{
//     name: string;
//     quantity: number;
//     price: number;
//   }>
// }
```

---

## Zod Methods

### String Methods

```typescript
z.string()
  .min(3, 'Min 3 characters')
  .max(20, 'Max 20 characters')
  .length(10, 'Exactly 10 characters')
  .email('Invalid email')
  .url('Invalid URL')
  .emoji('Must be emoji')
  .uuid('Invalid UUID')
  .regex(/pattern/, 'Invalid format')
  .startsWith('https://')
  .endsWith('.com')
  .includes('example')
  .trim()
  .toLowerCase()
  .toUpperCase()
```

---

### Number Methods

```typescript
z.number()
  .min(0)
  .max(100)
  .int()
  .positive()
  .negative()
  .nonnegative()
  .nonpositive()
  .multipleOf(5)
  .finite()
  .safe()
```

---

### Date Methods

```typescript
z.date()
  .min(new Date('2020-01-01'))
  .max(new Date('2025-12-31'))
```

---

### Array Methods

```typescript
z.array(z.string())
  .min(1)
  .max(10)
  .length(5)
  .nonempty()
```

---

### Object Methods

```typescript
z.object({ name: z.string() })
  .strict()  // No extra keys
  .partial()  // All fields optional
  .pick({ name: true })  // Pick specific fields
  .omit({ email: true })  // Omit specific fields
  .extend({ age: z.number() })  // Add fields
  .merge(otherSchema)  // Merge with another schema
```

---

## Error Handling

### Custom Error Messages

```typescript
const schema = z.object({
  email: z.string({
    required_error: 'Email is required',
    invalid_type_error: 'Email must be a string'
  }).email('Please enter a valid email address'),
  
  age: z.number({
    required_error: 'Age is required',
    invalid_type_error: 'Age must be a number'
  }).min(18, 'You must be at least 18 years old')
});
```

---

### Parse vs SafeParse

```typescript
const schema = z.object({
  name: z.string()
});

// parse() - throws error if invalid
try {
  const result = schema.parse({ name: 'John' });
  console.log(result);  // { name: 'John' }
} catch (error) {
  console.error(error);
}

// safeParse() - returns result object (recommended)
const result = schema.safeParse({ name: 'John' });

if (result.success) {
  console.log(result.data);  // { name: 'John' }
} else {
  console.log(result.error.errors);  // Array of errors
}
```

---

### Error Structure

```typescript
const schema = z.object({
  email: z.string().email(),
  age: z.number().min(18)
});

const result = schema.safeParse({ email: 'invalid', age: 10 });

if (!result.success) {
  console.log(result.error.errors);
  // [
  //   {
  //     code: 'invalid_string',
  //     validation: 'email',
  //     path: ['email'],
  //     message: 'Invalid email'
  //   },
  //   {
  //     code: 'too_small',
  //     minimum: 18,
  //     path: ['age'],
  //     message: 'Number must be greater than or equal to 18'
  //   }
  // ]
  
  // Formatted errors
  console.log(result.error.format());
  // {
  //   email: { _errors: ['Invalid email'] },
  //   age: { _errors: ['Number must be greater than or equal to 18'] }
  // }
}
```

---

## Complete Form Example

```typescript
import React from 'react';
import { useForm, useFieldArray } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Define schemas
const addressSchema = z.object({
  street: z.string().min(1, 'Street is required'),
  city: z.string().min(1, 'City is required'),
  state: z.string().min(2).max(2),
  zipCode: z.string().regex(/^\d{5}$/, 'Invalid zip code')
});

const phoneSchema = z.object({
  type: z.enum(['home', 'work', 'mobile']),
  number: z.string().regex(/^\d{10}$/, 'Must be 10 digits')
});

const profileSchema = z.object({
  firstName: z.string().min(2, 'Min 2 characters'),
  lastName: z.string().min(2, 'Min 2 characters'),
  email: z.string().email('Invalid email'),
  age: z.number().int().min(18, 'Must be 18+').max(120),
  gender: z.enum(['male', 'female', 'other']),
  address: addressSchema,
  phones: z.array(phoneSchema).min(1, 'At least one phone is required'),
  bio: z.string().min(10, 'Min 10 characters').max(500, 'Max 500 characters'),
  website: z.string().url('Invalid URL').optional().or(z.literal('')),
  acceptTerms: z.boolean().refine(val => val === true, {
    message: 'You must accept the terms'
  })
});

type ProfileFormData = z.infer<typeof profileSchema>;

function ProfileForm() {
  const {
    register,
    control,
    handleSubmit,
    formState: { errors }
  } = useForm<ProfileFormData>({
    resolver: zodResolver(profileSchema),
    defaultValues: {
      phones: [{ type: 'mobile', number: '' }]
    }
  });
  
  const { fields, append, remove } = useFieldArray({
    control,
    name: 'phones'
  });
  
  const onSubmit = (data: ProfileFormData) => {
    console.log('Form data:', data);
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <h1>Profile Form</h1>
      
      {/* Name */}
      <div>
        <input {...register('firstName')} placeholder="First Name" />
        {errors.firstName && <span>{errors.firstName.message}</span>}
      </div>
      
      <div>
        <input {...register('lastName')} placeholder="Last Name" />
        {errors.lastName && <span>{errors.lastName.message}</span>}
      </div>
      
      {/* Email */}
      <div>
        <input {...register('email')} placeholder="Email" />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      {/* Age */}
      <div>
        <input
          type="number"
          {...register('age', { valueAsNumber: true })}
          placeholder="Age"
        />
        {errors.age && <span>{errors.age.message}</span>}
      </div>
      
      {/* Gender */}
      <div>
        <select {...register('gender')}>
          <option value="">Select gender</option>
          <option value="male">Male</option>
          <option value="female">Female</option>
          <option value="other">Other</option>
        </select>
        {errors.gender && <span>{errors.gender.message}</span>}
      </div>
      
      {/* Address */}
      <fieldset>
        <legend>Address</legend>
        <input {...register('address.street')} placeholder="Street" />
        {errors.address?.street && <span>{errors.address.street.message}</span>}
        
        <input {...register('address.city')} placeholder="City" />
        {errors.address?.city && <span>{errors.address.city.message}</span>}
        
        <input {...register('address.state')} placeholder="State (2 letters)" />
        {errors.address?.state && <span>{errors.address.state.message}</span>}
        
        <input {...register('address.zipCode')} placeholder="Zip Code" />
        {errors.address?.zipCode && <span>{errors.address.zipCode.message}</span>}
      </fieldset>
      
      {/* Phones */}
      <fieldset>
        <legend>Phone Numbers</legend>
        {fields.map((field, index) => (
          <div key={field.id}>
            <select {...register(`phones.${index}.type`)}>
              <option value="mobile">Mobile</option>
              <option value="home">Home</option>
              <option value="work">Work</option>
            </select>
            
            <input
              {...register(`phones.${index}.number`)}
              placeholder="10 digits"
            />
            {errors.phones?.[index]?.number && (
              <span>{errors.phones[index]?.number?.message}</span>
            )}
            
            <button type="button" onClick={() => remove(index)}>Remove</button>
          </div>
        ))}
        <button type="button" onClick={() => append({ type: 'mobile', number: '' })}>
          Add Phone
        </button>
      </fieldset>
      
      {/* Bio */}
      <div>
        <textarea {...register('bio')} rows={5} placeholder="Bio" />
        {errors.bio && <span>{errors.bio.message}</span>}
      </div>
      
      {/* Website */}
      <div>
        <input {...register('website')} placeholder="Website (optional)" />
        {errors.website && <span>{errors.website.message}</span>}
      </div>
      
      {/* Terms */}
      <div>
        <label>
          <input type="checkbox" {...register('acceptTerms')} />
          I accept the terms and conditions
        </label>
        {errors.acceptTerms && <span>{errors.acceptTerms.message}</span>}
      </div>
      
      <button type="submit">Submit</button>
    </form>
  );
}

export default ProfileForm;
```

---

## Reusable Schemas

```typescript
// Create reusable schemas
const emailSchema = z.string().email('Invalid email');
const passwordSchema = z
  .string()
  .min(8, 'Min 8 characters')
  .regex(/[A-Z]/, 'Must contain uppercase')
  .regex(/[a-z]/, 'Must contain lowercase')
  .regex(/\d/, 'Must contain number');
const nameSchema = z.string().min(2).max(50);

// Compose schemas
const loginSchema = z.object({
  email: emailSchema,
  password: passwordSchema
});

const signupSchema = z.object({
  firstName: nameSchema,
  lastName: nameSchema,
  email: emailSchema,
  password: passwordSchema,
  confirmPassword: z.string()
}).refine(data => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword']
});

// Extend schemas
const userProfileSchema = signupSchema.extend({
  bio: z.string().optional(),
  avatar: z.string().url().optional()
});
```

---

## Best Practices

**1. Define schemas outside components**
```typescript
// ✅ Good - defined once
const schema = z.object({ /* ... */ });

function MyForm() {
  const { register } = useForm({ resolver: zodResolver(schema) });
}
```

**2. Use type inference**
```typescript
// ✅ Automatic type safety
type FormData = z.infer<typeof schema>;
```

**3. Use safeParse instead of parse**
```typescript
// ✅ Handles errors gracefully
const result = schema.safeParse(data);
```

**4. Provide clear error messages**
```typescript
z.string().min(8, 'Password must be at least 8 characters')
```

**5. Use enums for fixed values**
```typescript
z.enum(['admin', 'user', 'moderator'])
```

**6. Compose and reuse schemas**
```typescript
const addressSchema = z.object({ /* ... */ });
const userSchema = z.object({
  address: addressSchema
});
```

---

**Interview Tips:**
- Zod = **TypeScript-first** schema validation library
- Created by **Colin McDonnell**
- **Zero dependencies** (lightweight)
- **Type inference** with z.infer<typeof schema>
- **~8KB** minified (smaller than Yup)
- Import: **import { z } from 'zod'**
- **z.object()** defines object schema
- **z.string()** for strings
- **z.number()** for numbers
- **z.boolean()** for booleans
- **z.date()** for dates
- **z.array()** for arrays
- **z.enum()** for enums
- **String methods**: min, max, email, url, regex
- **Number methods**: min, max, int, positive
- **safeParse()** returns result object (recommended)
- **parse()** throws error if invalid
- **refine()** for custom validation
- **transform()** to modify values
- **optional()** makes field optional
- **nullable()** allows null
- **default()** sets default value
- **@hookform/resolvers/zod** for React Hook Form
- **zodResolver(schema)** in useForm
- **Cross-field validation** with refine on schema
- **Async validation** supported in refine
- **Error messages** customizable
- **Type-safe** end-to-end
- **Composable** schemas
- **extend()** to add fields
- **merge()** to combine schemas
- **pick()** and **omit()** to select fields
- **partial()** makes all fields optional
- **strict()** disallows extra keys
- Better **TypeScript** integration than Yup
- **Tree-shakeable** for smaller bundles
- Growing **ecosystem** and popularity
- Recommended for **new TypeScript projects**
- **Discriminated unions** for complex types
- **Native enum** support with z.nativeEnum()
- **Union types** with z.union()
- **Literal types** with z.literal()

</details>

---

### 77. How do you implement multi-step forms?

<details>
<summary>View Answer</summary>

**Multi-Step Forms**

Multi-step forms (also called wizard forms) break long forms into smaller, manageable steps. This improves user experience by reducing cognitive load and making forms less overwhelming.

**Common approaches:**
1. **State-based** - Store current step in state
2. **URL-based** - Use routing for each step
3. **Library-based** - Use form libraries with step support

---

## Basic Multi-Step Form

### Simple State-Based Approach

```jsx
import React, { useState } from 'react';

function MultiStepForm() {
  const [currentStep, setCurrentStep] = useState(1);
  const [formData, setFormData] = useState({
    // Step 1
    firstName: '',
    lastName: '',
    email: '',
    // Step 2
    address: '',
    city: '',
    zipCode: '',
    // Step 3
    cardNumber: '',
    expiryDate: '',
    cvv: ''
  });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };
  
  const nextStep = () => {
    setCurrentStep(prev => prev + 1);
  };
  
  const prevStep = () => {
    setCurrentStep(prev => prev - 1);
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted:', formData);
    // Submit to API
  };
  
  return (
    <div className="multi-step-form">
      <div className="progress-bar">
        <div className="progress" style={{ width: `${(currentStep / 3) * 100}%` }} />
      </div>
      
      <h2>Step {currentStep} of 3</h2>
      
      <form onSubmit={handleSubmit}>
        {currentStep === 1 && (
          <div className="step">
            <h3>Personal Information</h3>
            <input
              name="firstName"
              value={formData.firstName}
              onChange={handleChange}
              placeholder="First Name"
            />
            <input
              name="lastName"
              value={formData.lastName}
              onChange={handleChange}
              placeholder="Last Name"
            />
            <input
              name="email"
              type="email"
              value={formData.email}
              onChange={handleChange}
              placeholder="Email"
            />
            <button type="button" onClick={nextStep}>Next</button>
          </div>
        )}
        
        {currentStep === 2 && (
          <div className="step">
            <h3>Address</h3>
            <input
              name="address"
              value={formData.address}
              onChange={handleChange}
              placeholder="Street Address"
            />
            <input
              name="city"
              value={formData.city}
              onChange={handleChange}
              placeholder="City"
            />
            <input
              name="zipCode"
              value={formData.zipCode}
              onChange={handleChange}
              placeholder="Zip Code"
            />
            <button type="button" onClick={prevStep}>Back</button>
            <button type="button" onClick={nextStep}>Next</button>
          </div>
        )}
        
        {currentStep === 3 && (
          <div className="step">
            <h3>Payment</h3>
            <input
              name="cardNumber"
              value={formData.cardNumber}
              onChange={handleChange}
              placeholder="Card Number"
            />
            <input
              name="expiryDate"
              value={formData.expiryDate}
              onChange={handleChange}
              placeholder="MM/YY"
            />
            <input
              name="cvv"
              value={formData.cvv}
              onChange={handleChange}
              placeholder="CVV"
            />
            <button type="button" onClick={prevStep}>Back</button>
            <button type="submit">Submit</button>
          </div>
        )}
      </form>
    </div>
  );
}

export default MultiStepForm;
```

---

## Component-Based Multi-Step Form

**Better organization with separate step components:**

```jsx
import React, { useState } from 'react';

// Step 1 Component
function Step1({ formData, handleChange, nextStep }) {
  return (
    <div className="step">
      <h3>Personal Information</h3>
      <input
        name="firstName"
        value={formData.firstName}
        onChange={handleChange}
        placeholder="First Name"
        required
      />
      <input
        name="lastName"
        value={formData.lastName}
        onChange={handleChange}
        placeholder="Last Name"
        required
      />
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
        required
      />
      <button type="button" onClick={nextStep}>
        Next
      </button>
    </div>
  );
}

// Step 2 Component
function Step2({ formData, handleChange, nextStep, prevStep }) {
  return (
    <div className="step">
      <h3>Address</h3>
      <input
        name="address"
        value={formData.address}
        onChange={handleChange}
        placeholder="Street Address"
        required
      />
      <input
        name="city"
        value={formData.city}
        onChange={handleChange}
        placeholder="City"
        required
      />
      <input
        name="zipCode"
        value={formData.zipCode}
        onChange={handleChange}
        placeholder="Zip Code"
        required
      />
      <div className="button-group">
        <button type="button" onClick={prevStep}>
          Back
        </button>
        <button type="button" onClick={nextStep}>
          Next
        </button>
      </div>
    </div>
  );
}

// Step 3 Component
function Step3({ formData, handleChange, prevStep }) {
  return (
    <div className="step">
      <h3>Payment</h3>
      <input
        name="cardNumber"
        value={formData.cardNumber}
        onChange={handleChange}
        placeholder="Card Number"
        required
      />
      <input
        name="expiryDate"
        value={formData.expiryDate}
        onChange={handleChange}
        placeholder="MM/YY"
        required
      />
      <input
        name="cvv"
        value={formData.cvv}
        onChange={handleChange}
        placeholder="CVV"
        required
      />
      <div className="button-group">
        <button type="button" onClick={prevStep}>
          Back
        </button>
        <button type="submit">Submit</button>
      </div>
    </div>
  );
}

// Main Form Component
function MultiStepForm() {
  const [currentStep, setCurrentStep] = useState(1);
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    address: '',
    city: '',
    zipCode: '',
    cardNumber: '',
    expiryDate: '',
    cvv: ''
  });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const nextStep = () => setCurrentStep(prev => prev + 1);
  const prevStep = () => setCurrentStep(prev => prev - 1);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted:', formData);
    alert('Form submitted successfully!');
  };
  
  const steps = [
    { number: 1, title: 'Personal Info' },
    { number: 2, title: 'Address' },
    { number: 3, title: 'Payment' }
  ];
  
  return (
    <div className="multi-step-form">
      {/* Step Indicator */}
      <div className="step-indicator">
        {steps.map((step) => (
          <div
            key={step.number}
            className={`step-item ${currentStep >= step.number ? 'active' : ''}`}
          >
            <div className="step-number">{step.number}</div>
            <div className="step-title">{step.title}</div>
          </div>
        ))}
      </div>
      
      <form onSubmit={handleSubmit}>
        {currentStep === 1 && (
          <Step1
            formData={formData}
            handleChange={handleChange}
            nextStep={nextStep}
          />
        )}
        
        {currentStep === 2 && (
          <Step2
            formData={formData}
            handleChange={handleChange}
            nextStep={nextStep}
            prevStep={prevStep}
          />
        )}
        
        {currentStep === 3 && (
          <Step3
            formData={formData}
            handleChange={handleChange}
            prevStep={prevStep}
          />
        )}
      </form>
    </div>
  );
}

export default MultiStepForm;
```

---

## With Validation

**Add validation to each step:**

```jsx
import React, { useState } from 'react';

function MultiStepFormWithValidation() {
  const [currentStep, setCurrentStep] = useState(1);
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    address: '',
    city: '',
    zipCode: ''
  });
  const [errors, setErrors] = useState({});
  
  const validateStep1 = () => {
    const errors = {};
    
    if (!formData.firstName.trim()) {
      errors.firstName = 'First name is required';
    }
    
    if (!formData.lastName.trim()) {
      errors.lastName = 'Last name is required';
    }
    
    if (!formData.email.trim()) {
      errors.email = 'Email is required';
    } else if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(formData.email)) {
      errors.email = 'Invalid email address';
    }
    
    return errors;
  };
  
  const validateStep2 = () => {
    const errors = {};
    
    if (!formData.address.trim()) {
      errors.address = 'Address is required';
    }
    
    if (!formData.city.trim()) {
      errors.city = 'City is required';
    }
    
    if (!formData.zipCode.trim()) {
      errors.zipCode = 'Zip code is required';
    } else if (!/^\d{5}$/.test(formData.zipCode)) {
      errors.zipCode = 'Invalid zip code';
    }
    
    return errors;
  };
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Clear error when user types
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };
  
  const nextStep = () => {
    let validationErrors = {};
    
    if (currentStep === 1) {
      validationErrors = validateStep1();
    } else if (currentStep === 2) {
      validationErrors = validateStep2();
    }
    
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }
    
    setErrors({});
    setCurrentStep(prev => prev + 1);
  };
  
  const prevStep = () => {
    setCurrentStep(prev => prev - 1);
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted:', formData);
    alert('Form submitted successfully!');
  };
  
  return (
    <div className="multi-step-form">
      <h2>Step {currentStep} of 2</h2>
      
      <form onSubmit={handleSubmit}>
        {currentStep === 1 && (
          <div className="step">
            <h3>Personal Information</h3>
            
            <div className="form-group">
              <input
                name="firstName"
                value={formData.firstName}
                onChange={handleChange}
                placeholder="First Name"
                className={errors.firstName ? 'error' : ''}
              />
              {errors.firstName && <span className="error-message">{errors.firstName}</span>}
            </div>
            
            <div className="form-group">
              <input
                name="lastName"
                value={formData.lastName}
                onChange={handleChange}
                placeholder="Last Name"
                className={errors.lastName ? 'error' : ''}
              />
              {errors.lastName && <span className="error-message">{errors.lastName}</span>}
            </div>
            
            <div className="form-group">
              <input
                name="email"
                type="email"
                value={formData.email}
                onChange={handleChange}
                placeholder="Email"
                className={errors.email ? 'error' : ''}
              />
              {errors.email && <span className="error-message">{errors.email}</span>}
            </div>
            
            <button type="button" onClick={nextStep}>Next</button>
          </div>
        )}
        
        {currentStep === 2 && (
          <div className="step">
            <h3>Address</h3>
            
            <div className="form-group">
              <input
                name="address"
                value={formData.address}
                onChange={handleChange}
                placeholder="Street Address"
                className={errors.address ? 'error' : ''}
              />
              {errors.address && <span className="error-message">{errors.address}</span>}
            </div>
            
            <div className="form-group">
              <input
                name="city"
                value={formData.city}
                onChange={handleChange}
                placeholder="City"
                className={errors.city ? 'error' : ''}
              />
              {errors.city && <span className="error-message">{errors.city}</span>}
            </div>
            
            <div className="form-group">
              <input
                name="zipCode"
                value={formData.zipCode}
                onChange={handleChange}
                placeholder="Zip Code"
                className={errors.zipCode ? 'error' : ''}
              />
              {errors.zipCode && <span className="error-message">{errors.zipCode}</span>}
            </div>
            
            <div className="button-group">
              <button type="button" onClick={prevStep}>Back</button>
              <button type="submit">Submit</button>
            </div>
          </div>
        )}
      </form>
    </div>
  );
}

export default MultiStepFormWithValidation;
```

---

## With React Hook Form

```jsx
import React, { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Schemas for each step
const step1Schema = z.object({
  firstName: z.string().min(2, 'First name must be at least 2 characters'),
  lastName: z.string().min(2, 'Last name must be at least 2 characters'),
  email: z.string().email('Invalid email address')
});

const step2Schema = z.object({
  address: z.string().min(5, 'Address is required'),
  city: z.string().min(2, 'City is required'),
  zipCode: z.string().regex(/^\d{5}$/, 'Invalid zip code')
});

const step3Schema = z.object({
  cardNumber: z.string().regex(/^\d{16}$/, 'Card number must be 16 digits'),
  expiryDate: z.string().regex(/^\d{2}\/\d{2}$/, 'Format: MM/YY'),
  cvv: z.string().regex(/^\d{3,4}$/, 'CVV must be 3-4 digits')
});

// Combined schema
const completeSchema = step1Schema.merge(step2Schema).merge(step3Schema);

function MultiStepFormRHF() {
  const [currentStep, setCurrentStep] = useState(1);
  
  const {
    register,
    handleSubmit,
    trigger,
    formState: { errors }
  } = useForm({
    resolver: zodResolver(completeSchema),
    mode: 'onChange'
  });
  
  const nextStep = async () => {
    let fieldsToValidate = [];
    
    if (currentStep === 1) {
      fieldsToValidate = ['firstName', 'lastName', 'email'];
    } else if (currentStep === 2) {
      fieldsToValidate = ['address', 'city', 'zipCode'];
    }
    
    const isValid = await trigger(fieldsToValidate);
    
    if (isValid) {
      setCurrentStep(prev => prev + 1);
    }
  };
  
  const prevStep = () => {
    setCurrentStep(prev => prev - 1);
  };
  
  const onSubmit = (data) => {
    console.log('Form submitted:', data);
    alert('Form submitted successfully!');
  };
  
  return (
    <div className="multi-step-form">
      <h2>Step {currentStep} of 3</h2>
      
      <form onSubmit={handleSubmit(onSubmit)}>
        {currentStep === 1 && (
          <div className="step">
            <h3>Personal Information</h3>
            
            <div>
              <input {...register('firstName')} placeholder="First Name" />
              {errors.firstName && <span>{errors.firstName.message}</span>}
            </div>
            
            <div>
              <input {...register('lastName')} placeholder="Last Name" />
              {errors.lastName && <span>{errors.lastName.message}</span>}
            </div>
            
            <div>
              <input {...register('email')} placeholder="Email" />
              {errors.email && <span>{errors.email.message}</span>}
            </div>
            
            <button type="button" onClick={nextStep}>Next</button>
          </div>
        )}
        
        {currentStep === 2 && (
          <div className="step">
            <h3>Address</h3>
            
            <div>
              <input {...register('address')} placeholder="Street Address" />
              {errors.address && <span>{errors.address.message}</span>}
            </div>
            
            <div>
              <input {...register('city')} placeholder="City" />
              {errors.city && <span>{errors.city.message}</span>}
            </div>
            
            <div>
              <input {...register('zipCode')} placeholder="Zip Code" />
              {errors.zipCode && <span>{errors.zipCode.message}</span>}
            </div>
            
            <button type="button" onClick={prevStep}>Back</button>
            <button type="button" onClick={nextStep}>Next</button>
          </div>
        )}
        
        {currentStep === 3 && (
          <div className="step">
            <h3>Payment</h3>
            
            <div>
              <input {...register('cardNumber')} placeholder="Card Number" />
              {errors.cardNumber && <span>{errors.cardNumber.message}</span>}
            </div>
            
            <div>
              <input {...register('expiryDate')} placeholder="MM/YY" />
              {errors.expiryDate && <span>{errors.expiryDate.message}</span>}
            </div>
            
            <div>
              <input {...register('cvv')} placeholder="CVV" />
              {errors.cvv && <span>{errors.cvv.message}</span>}
            </div>
            
            <button type="button" onClick={prevStep}>Back</button>
            <button type="submit">Submit</button>
          </div>
        )}
      </form>
    </div>
  );
}

export default MultiStepFormRHF;
```

---

## With Progress Indicator

```jsx
import React, { useState } from 'react';

function MultiStepFormWithProgress() {
  const [currentStep, setCurrentStep] = useState(1);
  const totalSteps = 4;
  const [formData, setFormData] = useState({
    // Step data
  });
  
  const steps = [
    { number: 1, title: 'Account', icon: '👤' },
    { number: 2, title: 'Profile', icon: '📝' },
    { number: 3, title: 'Preferences', icon: '⚙️' },
    { number: 4, title: 'Review', icon: '✓' }
  ];
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const nextStep = () => setCurrentStep(prev => Math.min(prev + 1, totalSteps));
  const prevStep = () => setCurrentStep(prev => Math.max(prev - 1, 1));
  const goToStep = (step) => setCurrentStep(step);
  
  return (
    <div className="multi-step-form-container">
      {/* Horizontal Step Indicator */}
      <div className="step-indicator-horizontal">
        {steps.map((step, index) => (
          <div key={step.number} className="step-wrapper">
            <div
              className={`step-circle ${
                currentStep >= step.number ? 'active' : ''
              } ${currentStep === step.number ? 'current' : ''}`}
              onClick={() => goToStep(step.number)}
            >
              <span className="step-icon">{step.icon}</span>
              <span className="step-num">{step.number}</span>
            </div>
            <div className="step-label">{step.title}</div>
            {index < steps.length - 1 && (
              <div className={`step-line ${currentStep > step.number ? 'active' : ''}`} />
            )}
          </div>
        ))}
      </div>
      
      {/* Progress Bar */}
      <div className="progress-bar-container">
        <div
          className="progress-bar-fill"
          style={{ width: `${(currentStep / totalSteps) * 100}%` }}
        />
      </div>
      
      {/* Form Content */}
      <div className="form-content">
        {/* Step content here */}
        <h2>Step {currentStep}: {steps[currentStep - 1].title}</h2>
        
        {/* Navigation Buttons */}
        <div className="navigation-buttons">
          {currentStep > 1 && (
            <button type="button" onClick={prevStep}>
              ← Previous
            </button>
          )}
          {currentStep < totalSteps && (
            <button type="button" onClick={nextStep}>
              Next →
            </button>
          )}
          {currentStep === totalSteps && (
            <button type="submit">
              Submit
            </button>
          )}
        </div>
      </div>
    </div>
  );
}

export default MultiStepFormWithProgress;
```

**CSS for Progress Indicator:**
```css
.step-indicator-horizontal {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  position: relative;
}

.step-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  flex: 1;
}

.step-circle {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: #e0e0e0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  z-index: 2;
}

.step-circle.active {
  background: #4CAF50;
  color: white;
}

.step-circle.current {
  border: 3px solid #4CAF50;
  box-shadow: 0 0 0 4px rgba(76, 175, 80, 0.2);
}

.step-label {
  margin-top: 8px;
  font-size: 12px;
  font-weight: 500;
  color: #666;
}

.step-line {
  position: absolute;
  top: 25px;
  left: 50%;
  width: 100%;
  height: 2px;
  background: #e0e0e0;
  z-index: 1;
}

.step-line.active {
  background: #4CAF50;
}

.progress-bar-container {
  width: 100%;
  height: 8px;
  background: #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 30px;
}

.progress-bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #4CAF50, #45a049);
  transition: width 0.3s ease;
}
```

---

## With URL/Routing

**Use React Router for step navigation:**

```jsx
import React, { useState } from 'react';
import { useNavigate, useParams, Routes, Route } from 'react-router-dom';

function Step1({ formData, updateFormData }) {
  const navigate = useNavigate();
  
  const handleNext = () => {
    navigate('/signup/step2');
  };
  
  return (
    <div>
      <h3>Step 1: Personal Info</h3>
      <input
        name="firstName"
        value={formData.firstName}
        onChange={(e) => updateFormData('firstName', e.target.value)}
        placeholder="First Name"
      />
      <button onClick={handleNext}>Next</button>
    </div>
  );
}

function Step2({ formData, updateFormData }) {
  const navigate = useNavigate();
  
  return (
    <div>
      <h3>Step 2: Address</h3>
      <input
        name="address"
        value={formData.address}
        onChange={(e) => updateFormData('address', e.target.value)}
        placeholder="Address"
      />
      <button onClick={() => navigate('/signup/step1')}>Back</button>
      <button onClick={() => navigate('/signup/step3')}>Next</button>
    </div>
  );
}

function Step3({ formData, handleSubmit }) {
  const navigate = useNavigate();
  
  return (
    <div>
      <h3>Step 3: Review</h3>
      <p>Name: {formData.firstName}</p>
      <p>Address: {formData.address}</p>
      <button onClick={() => navigate('/signup/step2')}>Back</button>
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}

function MultiStepFormRouted() {
  const [formData, setFormData] = useState({
    firstName: '',
    address: ''
  });
  
  const updateFormData = (field, value) => {
    setFormData(prev => ({ ...prev, [field]: value }));
  };
  
  const handleSubmit = () => {
    console.log('Form submitted:', formData);
    alert('Form submitted!');
  };
  
  return (
    <div>
      <Routes>
        <Route
          path="/step1"
          element={<Step1 formData={formData} updateFormData={updateFormData} />}
        />
        <Route
          path="/step2"
          element={<Step2 formData={formData} updateFormData={updateFormData} />}
        />
        <Route
          path="/step3"
          element={<Step3 formData={formData} handleSubmit={handleSubmit} />}
        />
      </Routes>
    </div>
  );
}

export default MultiStepFormRouted;
```

---

## Save Progress (LocalStorage)

```jsx
import React, { useState, useEffect } from 'react';

function MultiStepFormWithSave() {
  const [currentStep, setCurrentStep] = useState(1);
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: ''
  });
  
  // Load saved data on mount
  useEffect(() => {
    const savedData = localStorage.getItem('multiStepFormData');
    const savedStep = localStorage.getItem('multiStepFormStep');
    
    if (savedData) {
      setFormData(JSON.parse(savedData));
    }
    
    if (savedStep) {
      setCurrentStep(parseInt(savedStep, 10));
    }
  }, []);
  
  // Save data on change
  useEffect(() => {
    localStorage.setItem('multiStepFormData', JSON.stringify(formData));
    localStorage.setItem('multiStepFormStep', currentStep.toString());
  }, [formData, currentStep]);
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted:', formData);
    
    // Clear saved data after successful submission
    localStorage.removeItem('multiStepFormData');
    localStorage.removeItem('multiStepFormStep');
    
    alert('Form submitted successfully!');
  };
  
  return (
    <div>
      <p>Your progress is automatically saved</p>
      <form onSubmit={handleSubmit}>
        {/* Form steps */}
      </form>
    </div>
  );
}

export default MultiStepFormWithSave;
```

---

## Best Practices

**1. Maintain form state across steps**
```jsx
// Store all form data in one state object
const [formData, setFormData] = useState({
  step1Field: '',
  step2Field: '',
  step3Field: ''
});
```

**2. Validate before moving to next step**
```jsx
const nextStep = () => {
  const errors = validateCurrentStep();
  if (Object.keys(errors).length === 0) {
    setCurrentStep(prev => prev + 1);
  }
};
```

**3. Show progress indicator**
```jsx
<div className="progress">
  Step {currentStep} of {totalSteps}
</div>
```

**4. Allow navigation to previous steps**
```jsx
<button onClick={() => setCurrentStep(prev => prev - 1)}>
  Back
</button>
```

**5. Save progress to prevent data loss**
```jsx
// Use localStorage or session storage
localStorage.setItem('formData', JSON.stringify(formData));
```

**6. Disable Next button when invalid**
```jsx
<button onClick={nextStep} disabled={!isStepValid}>
  Next
</button>
```

**7. Show summary before submission**
```jsx
// Final step shows all entered data for review
```

---

**Interview Tips:**
- Multi-step forms = **wizard forms** breaking long forms into steps
- Improves **user experience** (less overwhelming)
- Reduces **cognitive load**
- Better for **mobile** devices
- Store **currentStep** in state
- Store **all form data** in single state object
- **Validate each step** before proceeding
- **Progress indicator** shows completion
- Allow **navigation back** to previous steps
- **Save progress** to localStorage/sessionStorage
- Consider **URL-based** steps with routing
- Use **React Hook Form** trigger() for validation
- **Component-based** approach for organization
- Show **summary/review** step before submission
- **Disable Next** button when step is invalid
- Clear saved data after **successful submission**
- Use **schema validation** (Yup/Zod) per step
- **Conditional steps** based on previous answers
- **Animation** between steps improves UX
- **Mobile-friendly** navigation
- **Auto-save** drafts periodically
- Common patterns: **registration**, **checkout**, **onboarding**
- **Track analytics** per step (completion rates)
- **Error handling** per step
- **Loading states** during submission
- **Success message** after completion
- **Edit previous steps** without losing data

</details>

---

### 78. What are controlled vs uncontrolled forms?

<details>
<summary>View Answer</summary>

**Controlled vs Uncontrolled Components**

In React, form inputs can be either **controlled** (React controls the value) or **uncontrolled** (DOM controls the value).

---

## Controlled Components

**Definition:** Form inputs whose values are controlled by React state.

**How it works:**
- Value is stored in React state
- onChange handler updates state
- Component re-renders with new value
- Single source of truth (React state)

### Basic Example

```jsx
import React, { useState } from 'react';

function ControlledForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({ name, email });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      
      <button type="submit">Submit</button>
    </form>
  );
}

export default ControlledForm;
```

**Characteristics:**
- ✅ React controls the value
- ✅ Value stored in state
- ✅ onChange updates state
- ✅ Can validate in real-time
- ✅ Can format input immediately
- ✅ Single source of truth
- ❌ More boilerplate code
- ❌ More re-renders

---

### Multiple Inputs (Controlled)

```jsx
import React, { useState } from 'react';

function ControlledFormMultiple() {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    age: '',
    gender: '',
    terms: false
  });
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form data:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="firstName"
        value={formData.firstName}
        onChange={handleChange}
        placeholder="First Name"
      />
      
      <input
        name="lastName"
        value={formData.lastName}
        onChange={handleChange}
        placeholder="Last Name"
      />
      
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      
      <input
        name="age"
        type="number"
        value={formData.age}
        onChange={handleChange}
        placeholder="Age"
      />
      
      <select name="gender" value={formData.gender} onChange={handleChange}>
        <option value="">Select Gender</option>
        <option value="male">Male</option>
        <option value="female">Female</option>
        <option value="other">Other</option>
      </select>
      
      <label>
        <input
          name="terms"
          type="checkbox"
          checked={formData.terms}
          onChange={handleChange}
        />
        I accept the terms
      </label>
      
      <button type="submit">Submit</button>
    </form>
  );
}

export default ControlledFormMultiple;
```

---

### Real-time Validation (Controlled)

```jsx
import React, { useState } from 'react';

function ControlledWithValidation() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');
  
  const handleChange = (e) => {
    const value = e.target.value;
    setEmail(value);
    
    // Validate in real-time
    if (value && !/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(value)) {
      setError('Invalid email address');
    } else {
      setError('');
    }
  };
  
  return (
    <div>
      <input
        type="email"
        value={email}
        onChange={handleChange}
        placeholder="Email"
        className={error ? 'invalid' : ''}
      />
      {error && <span className="error">{error}</span>}
    </div>
  );
}
```

---

### Format Input (Controlled)

```jsx
import React, { useState } from 'react';

function PhoneInput() {
  const [phone, setPhone] = useState('');
  
  const formatPhoneNumber = (value) => {
    // Remove non-digits
    const cleaned = value.replace(/\D/g, '');
    
    // Format as (XXX) XXX-XXXX
    const match = cleaned.match(/^(\d{0,3})(\d{0,3})(\d{0,4})$/);
    
    if (!match) return value;
    
    if (match[3]) {
      return `(${match[1]}) ${match[2]}-${match[3]}`;
    } else if (match[2]) {
      return `(${match[1]}) ${match[2]}`;
    } else if (match[1]) {
      return `(${match[1]}`;
    }
    
    return '';
  };
  
  const handleChange = (e) => {
    const formatted = formatPhoneNumber(e.target.value);
    setPhone(formatted);
  };
  
  return (
    <input
      type="tel"
      value={phone}
      onChange={handleChange}
      placeholder="(555) 123-4567"
    />
  );
}
```

---

## Uncontrolled Components

**Definition:** Form inputs whose values are controlled by the DOM (like traditional HTML forms).

**How it works:**
- Value is NOT stored in React state
- Use refs to access values
- DOM is the source of truth
- Less React involvement

### Basic Example

```jsx
import React, { useRef } from 'react';

function UncontrolledForm() {
  const nameRef = useRef(null);
  const emailRef = useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const formData = {
      name: nameRef.current.value,
      email: emailRef.current.value
    };
    
    console.log(formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        ref={nameRef}
        defaultValue=""
        placeholder="Name"
      />
      
      <input
        type="email"
        ref={emailRef}
        defaultValue=""
        placeholder="Email"
      />
      
      <button type="submit">Submit</button>
    </form>
  );
}

export default UncontrolledForm;
```

**Characteristics:**
- ✅ Less boilerplate code
- ✅ Fewer re-renders
- ✅ Better performance
- ✅ Simpler for simple forms
- ❌ DOM is source of truth
- ❌ Harder to validate in real-time
- ❌ Harder to format input
- ❌ Less React-like

---

### With Default Values

```jsx
function UncontrolledWithDefaults() {
  const nameRef = useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(nameRef.current.value);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Use defaultValue, not value */}
      <input
        ref={nameRef}
        defaultValue="John Doe"
        placeholder="Name"
      />
      
      <select ref={useRef(null)} defaultValue="option2">
        <option value="option1">Option 1</option>
        <option value="option2">Option 2</option>
        <option value="option3">Option 3</option>
      </select>
      
      <textarea
        ref={useRef(null)}
        defaultValue="Default text"
      />
      
      <input
        type="checkbox"
        ref={useRef(null)}
        defaultChecked={true}
      />
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

### File Input (Always Uncontrolled)

```jsx
import React, { useRef } from 'react';

function FileUploadUncontrolled() {
  const fileInputRef = useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const file = fileInputRef.current.files[0];
    
    if (file) {
      console.log('File:', file.name);
      console.log('Size:', file.size);
      console.log('Type:', file.type);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* File inputs are ALWAYS uncontrolled */}
      <input
        type="file"
        ref={fileInputRef}
      />
      
      <button type="submit">Upload</button>
    </form>
  );
}
```

**Note:** File inputs are **always uncontrolled** because their value can only be set by the user for security reasons.

---

### Using FormData (Uncontrolled)

```jsx
import React from 'react';

function UncontrolledWithFormData() {
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Get all form data at once
    const formData = new FormData(e.target);
    
    // Convert to object
    const data = Object.fromEntries(formData);
    
    console.log(data);
    // { name: 'John', email: 'john@example.com', age: '25' }
    
    // Or iterate through entries
    for (let [key, value] of formData.entries()) {
      console.log(key, value);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="name" defaultValue="" placeholder="Name" />
      <input name="email" type="email" defaultValue="" placeholder="Email" />
      <input name="age" type="number" defaultValue="" placeholder="Age" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Comparison Table

| Feature | Controlled | Uncontrolled |
|---------|-----------|--------------|
| **Value Source** | React state | DOM |
| **Access Value** | From state | From ref |
| **Updates** | onChange handler | Not needed |
| **Re-renders** | On every change | Only on submit |
| **Validation** | Real-time | On submit |
| **Format Input** | Easy | Difficult |
| **Initial Value** | `value` prop | `defaultValue` prop |
| **Checkbox** | `checked` prop | `defaultChecked` prop |
| **Code** | More verbose | Less verbose |
| **Performance** | More re-renders | Fewer re-renders |
| **Testing** | Easier | Harder |
| **React Way** | ✅ Recommended | ⚠️ Use sparingly |
| **Use Case** | Most forms | Simple forms, file inputs |

---

## When to Use Which?

### Use Controlled Components When:

✅ **Real-time validation** needed
```jsx
// Show error as user types
if (password.length < 8) {
  setError('Password too short');
}
```

✅ **Format input** on the fly
```jsx
// Format phone number as user types
setPhone(formatPhoneNumber(value));
```

✅ **Conditional enabling/disabling**
```jsx
// Enable submit only when form is valid
<button disabled={!isValid}>Submit</button>
```

✅ **Dynamic form fields**
```jsx
// Show/hide fields based on input
{showAddress && <AddressFields />}
```

✅ **Multi-step forms**
```jsx
// Need to access values across steps
```

✅ **Instant feedback**
```jsx
// Character counter, password strength
```

---

### Use Uncontrolled Components When:

✅ **Simple forms** with few fields
```jsx
// Just need value on submit
```

✅ **File uploads**
```jsx
// File inputs are always uncontrolled
<input type="file" ref={fileRef} />
```

✅ **Performance critical**
```jsx
// Avoid re-renders on every keystroke
```

✅ **Third-party libraries**
```jsx
// Some libraries expect uncontrolled inputs
```

✅ **Non-React code integration**
```jsx
// Working with jQuery or vanilla JS
```

---

## Hybrid Approach

**Combine both for optimal solution:**

```jsx
import React, { useState, useRef } from 'react';

function HybridForm() {
  // Controlled for validation-critical fields
  const [email, setEmail] = useState('');
  const [emailError, setEmailError] = useState('');
  
  // Uncontrolled for other fields
  const nameRef = useRef(null);
  const messageRef = useRef(null);
  
  const handleEmailChange = (e) => {
    const value = e.target.value;
    setEmail(value);
    
    // Validate email in real-time
    if (value && !/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(value)) {
      setEmailError('Invalid email');
    } else {
      setEmailError('');
    }
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const formData = {
      name: nameRef.current.value,
      email: email,
      message: messageRef.current.value
    };
    
    console.log(formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Uncontrolled - no validation needed */}
      <input
        ref={nameRef}
        placeholder="Name"
      />
      
      {/* Controlled - needs real-time validation */}
      <input
        type="email"
        value={email}
        onChange={handleEmailChange}
        placeholder="Email"
        className={emailError ? 'invalid' : ''}
      />
      {emailError && <span className="error">{emailError}</span>}
      
      {/* Uncontrolled - no validation needed */}
      <textarea
        ref={messageRef}
        placeholder="Message"
      />
      
      <button type="submit" disabled={!!emailError}>
        Submit
      </button>
    </form>
  );
}

export default HybridForm;
```

---

## React Hook Form (Uncontrolled by Default)

**React Hook Form uses uncontrolled inputs by default for better performance:**

```jsx
import { useForm } from 'react-hook-form';

function RHFExample() {
  const { register, handleSubmit } = useForm();
  
  const onSubmit = (data) => {
    console.log(data);
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Uncontrolled with refs */}
      <input {...register('name')} placeholder="Name" />
      <input {...register('email')} placeholder="Email" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**React Hook Form advantages:**
- Uses uncontrolled components (refs)
- Minimal re-renders
- Better performance
- Still provides validation

---

## Common Patterns

### Pattern 1: Controlled Input with Debouncing

```jsx
import React, { useState, useEffect } from 'react';

function DebouncedSearch() {
  const [query, setQuery] = useState('');
  const [debouncedQuery, setDebouncedQuery] = useState('');
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedQuery(query);
    }, 500);
    
    return () => clearTimeout(timer);
  }, [query]);
  
  useEffect(() => {
    if (debouncedQuery) {
      console.log('Search for:', debouncedQuery);
      // API call here
    }
  }, [debouncedQuery]);
  
  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

---

### Pattern 2: Controlled Select with Dependent Fields

```jsx
import React, { useState } from 'react';

function DependentSelects() {
  const [country, setCountry] = useState('');
  const [state, setState] = useState('');
  
  const states = {
    USA: ['California', 'Texas', 'New York'],
    UK: ['England', 'Scotland', 'Wales'],
    Canada: ['Ontario', 'Quebec', 'British Columbia']
  };
  
  return (
    <form>
      <select value={country} onChange={(e) => {
        setCountry(e.target.value);
        setState('');  // Reset state when country changes
      }}>
        <option value="">Select Country</option>
        <option value="USA">USA</option>
        <option value="UK">UK</option>
        <option value="Canada">Canada</option>
      </select>
      
      {country && (
        <select value={state} onChange={(e) => setState(e.target.value)}>
          <option value="">Select State</option>
          {states[country].map(s => (
            <option key={s} value={s}>{s}</option>
          ))}
        </select>
      )}
    </form>
  );
}
```

---

### Pattern 3: Uncontrolled with Reset

```jsx
import React, { useRef } from 'react';

function UncontrolledWithReset() {
  const formRef = useRef(null);
  const nameRef = useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(nameRef.current.value);
    
    // Reset form
    formRef.current.reset();
  };
  
  const handleReset = () => {
    formRef.current.reset();
  };
  
  return (
    <form ref={formRef} onSubmit={handleSubmit}>
      <input ref={nameRef} name="name" placeholder="Name" />
      <button type="submit">Submit</button>
      <button type="button" onClick={handleReset}>Reset</button>
    </form>
  );
}
```

---

## Best Practices

**1. Prefer controlled components in React**
```jsx
// ✅ React way
const [value, setValue] = useState('');
<input value={value} onChange={(e) => setValue(e.target.value)} />
```

**2. Use uncontrolled for file inputs**
```jsx
// ✅ File inputs must be uncontrolled
<input type="file" ref={fileRef} />
```

**3. Use defaultValue for uncontrolled, value for controlled**
```jsx
// ✅ Uncontrolled
<input defaultValue="default" ref={ref} />

// ✅ Controlled
<input value={value} onChange={handleChange} />
```

**4. Don't mix controlled and uncontrolled**
```jsx
// ❌ Bad - switching between controlled/uncontrolled
const [value, setValue] = useState(null);
<input value={value} onChange={(e) => setValue(e.target.value)} />

// ✅ Good - always controlled
const [value, setValue] = useState('');
```

**5. Use React Hook Form for complex forms**
```jsx
// Uncontrolled with validation
const { register, handleSubmit } = useForm();
```

---

**Interview Tips:**
- **Controlled** = React controls value (via state)
- **Uncontrolled** = DOM controls value (via refs)
- Controlled uses **value** prop + **onChange** handler
- Uncontrolled uses **defaultValue** prop + **ref**
- Controlled = **single source of truth** (React state)
- Uncontrolled = **DOM is source of truth**
- Controlled allows **real-time validation**
- Controlled allows **input formatting**
- Controlled more **re-renders**
- Uncontrolled **better performance**
- **File inputs** are always uncontrolled
- **defaultValue** for initial value (uncontrolled)
- **defaultChecked** for checkboxes (uncontrolled)
- Access uncontrolled value with **ref.current.value**
- Use **FormData** API with uncontrolled
- **React Hook Form** uses uncontrolled by default
- **Formik** uses controlled approach
- Most React apps prefer **controlled**
- Use uncontrolled for **simple forms**
- **Hybrid approach** = use both
- Controlled = more **React-like**
- Don't **switch between** controlled/uncontrolled
- **null/undefined** value makes input uncontrolled
- Always start with **empty string** ''
- **Testing** easier with controlled
- **Third-party libraries** may expect uncontrolled
- Controlled = **instant feedback**
- Uncontrolled = **less boilerplate**
- Choose based on **use case**
- **Modern React** prefers controlled

</details>

---

### 79. How do you handle form submission in React?

<details>
<summary>View Answer</summary>

**Form Submission in React**

Form submission in React requires preventing default browser behavior and handling the submission with JavaScript.

---

## Basic Form Submission

### Prevent Default Behavior

```jsx
import React, { useState } from 'react';

function BasicForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  
  const handleSubmit = (e) => {
    // Prevent default form submission (page reload)
    e.preventDefault();
    
    console.log('Form submitted:', { name, email });
    
    // Submit to API
    // fetch('/api/submit', { ... });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <button type="submit">Submit</button>
    </form>
  );
}

export default BasicForm;
```

**Key points:**
- Use `onSubmit` on `<form>` element
- Call `e.preventDefault()` to prevent page reload
- Access form data from state
- `type="submit"` triggers form submission

---

## Submission with Validation

```jsx
import React, { useState } from 'react';

function FormWithValidation() {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const validate = () => {
    const errors = {};
    
    if (!formData.email) {
      errors.email = 'Email is required';
    } else if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(formData.email)) {
      errors.email = 'Invalid email address';
    }
    
    if (!formData.password) {
      errors.password = 'Password is required';
    } else if (formData.password.length < 8) {
      errors.password = 'Password must be at least 8 characters';
    }
    
    return errors;
  };
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Validate
    const validationErrors = validate();
    setErrors(validationErrors);
    
    // Stop if errors
    if (Object.keys(validationErrors).length > 0) {
      return;
    }
    
    // Submit
    setIsSubmitting(true);
    
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(formData)
      });
      
      if (response.ok) {
        const data = await response.json();
        console.log('Success:', data);
        // Redirect or show success message
      } else {
        setErrors({ submit: 'Login failed' });
      }
    } catch (error) {
      console.error('Error:', error);
      setErrors({ submit: 'Network error' });
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <input
          name="password"
          type="password"
          value={formData.password}
          onChange={handleChange}
          placeholder="Password"
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>
      
      {errors.submit && <div className="error">{errors.submit}</div>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}

export default FormWithValidation;
```

---

## Using FormData API

**Extract form data without state:**

```jsx
import React from 'react';

function FormDataExample() {
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Get FormData from form element
    const formData = new FormData(e.target);
    
    // Convert to object
    const data = Object.fromEntries(formData);
    console.log(data);
    // { name: 'John', email: 'john@example.com', age: '25' }
    
    // Or send FormData directly (multipart/form-data)
    await fetch('/api/submit', {
      method: 'POST',
      body: formData  // Don't set Content-Type, browser sets it automatically
    });
    
    // Or convert to JSON
    await fetch('/api/submit', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)
    });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="name" placeholder="Name" />
      <input name="email" type="email" placeholder="Email" />
      <input name="age" type="number" placeholder="Age" />
      <button type="submit">Submit</button>
    </form>
  );
}

export default FormDataExample;
```

---

## With React Hook Form

```jsx
import React from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Min 8 characters')
});

function RHFSubmission() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    reset,
    setError
  } = useForm({
    resolver: zodResolver(schema)
  });
  
  const onSubmit = async (data) => {
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });
      
      if (!response.ok) {
        const error = await response.json();
        
        // Set server-side errors
        if (error.field) {
          setError(error.field, {
            type: 'server',
            message: error.message
          });
        } else {
          setError('root', {
            type: 'server',
            message: error.message
          });
        }
        return;
      }
      
      const result = await response.json();
      console.log('Success:', result);
      
      // Reset form on success
      reset();
    } catch (error) {
      console.error('Error:', error);
      setError('root', {
        type: 'server',
        message: 'Network error'
      });
    }
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('email')} placeholder="Email" />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      <div>
        <input type="password" {...register('password')} placeholder="Password" />
        {errors.password && <span>{errors.password.message}</span>}
      </div>
      
      {errors.root && <div className="error">{errors.root.message}</div>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}

export default RHFSubmission;
```

---

## With Formik

```jsx
import React from 'react';
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

const validationSchema = Yup.object().shape({
  email: Yup.string().email('Invalid email').required('Required'),
  password: Yup.string().min(8, 'Min 8 characters').required('Required')
});

function FormikSubmission() {
  const handleSubmit = async (values, { setSubmitting, setErrors, resetForm }) => {
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(values)
      });
      
      if (!response.ok) {
        const error = await response.json();
        setErrors({ submit: error.message });
        return;
      }
      
      const data = await response.json();
      console.log('Success:', data);
      
      // Reset form
      resetForm();
    } catch (error) {
      console.error('Error:', error);
      setErrors({ submit: 'Network error' });
    } finally {
      setSubmitting(false);
    }
  };
  
  return (
    <Formik
      initialValues={{ email: '', password: '' }}
      validationSchema={validationSchema}
      onSubmit={handleSubmit}
    >
      {({ errors, isSubmitting }) => (
        <Form>
          <div>
            <Field name="email" type="email" placeholder="Email" />
            <ErrorMessage name="email" component="div" className="error" />
          </div>
          
          <div>
            <Field name="password" type="password" placeholder="Password" />
            <ErrorMessage name="password" component="div" className="error" />
          </div>
          
          {errors.submit && <div className="error">{errors.submit}</div>}
          
          <button type="submit" disabled={isSubmitting}>
            {isSubmitting ? 'Submitting...' : 'Submit'}
          </button>
        </Form>
      )}
    </Formik>
  );
}

export default FormikSubmission;
```

---

## Async Submission with Loading State

```jsx
import React, { useState } from 'react';

function AsyncSubmission() {
  const [formData, setFormData] = useState({ email: '', password: '' });
  const [status, setStatus] = useState('idle'); // idle | submitting | success | error
  const [message, setMessage] = useState('');
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setStatus('submitting');
    setMessage('');
    
    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      const response = await fetch('/api/submit', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });
      
      if (response.ok) {
        setStatus('success');
        setMessage('Form submitted successfully!');
        setFormData({ email: '', password: '' });
      } else {
        setStatus('error');
        setMessage('Submission failed. Please try again.');
      }
    } catch (error) {
      setStatus('error');
      setMessage('Network error. Please try again.');
    }
  };
  
  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          name="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Email"
          disabled={status === 'submitting'}
        />
        
        <input
          name="password"
          type="password"
          value={formData.password}
          onChange={handleChange}
          placeholder="Password"
          disabled={status === 'submitting'}
        />
        
        <button type="submit" disabled={status === 'submitting'}>
          {status === 'submitting' ? 'Submitting...' : 'Submit'}
        </button>
      </form>
      
      {status === 'success' && (
        <div className="success-message">{message}</div>
      )}
      
      {status === 'error' && (
        <div className="error-message">{message}</div>
      )}
    </div>
  );
}

export default AsyncSubmission;
```

---

## Submit on Enter Key

```jsx
function SubmitOnEnter() {
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted');
  };
  
  const handleKeyDown = (e) => {
    // Submit on Enter (without Shift)
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      e.target.form.requestSubmit();
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="name" placeholder="Name" />
      
      {/* Textarea - Shift+Enter for new line, Enter to submit */}
      <textarea
        name="message"
        placeholder="Message"
        onKeyDown={handleKeyDown}
      />
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Prevent Double Submission

```jsx
import React, { useState, useRef } from 'react';

function PreventDoubleSubmit() {
  const [isSubmitting, setIsSubmitting] = useState(false);
  const submitTimeoutRef = useRef(null);
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Prevent double submission
    if (isSubmitting) {
      return;
    }
    
    setIsSubmitting(true);
    
    try {
      await fetch('/api/submit', {
        method: 'POST',
        body: JSON.stringify({ /* data */ })
      });
      
      console.log('Submitted successfully');
    } catch (error) {
      console.error('Submission error:', error);
    } finally {
      // Re-enable after delay to prevent accidental double-clicks
      submitTimeoutRef.current = setTimeout(() => {
        setIsSubmitting(false);
      }, 1000);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

---

## Submit with File Upload

```jsx
import React, { useState } from 'react';

function FileUploadSubmit() {
  const [file, setFile] = useState(null);
  const [formData, setFormData] = useState({ name: '', email: '' });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleFileChange = (e) => {
    setFile(e.target.files[0]);
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Create FormData for file upload
    const data = new FormData();
    data.append('name', formData.name);
    data.append('email', formData.email);
    
    if (file) {
      data.append('file', file);
    }
    
    try {
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: data  // Don't set Content-Type header
      });
      
      if (response.ok) {
        console.log('Uploaded successfully');
      }
    } catch (error) {
      console.error('Upload error:', error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Name"
      />
      
      <input
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      
      <input
        type="file"
        onChange={handleFileChange}
      />
      
      <button type="submit">Submit</button>
    </form>
  );
}

export default FileUploadSubmit;
```

---

## Reset Form After Submission

```jsx
import React, { useState } from 'react';

function ResetForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: ''
  });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    console.log('Submitting:', formData);
    
    // Submit to API
    await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(formData)
    });
    
    // Reset form
    setFormData({ name: '', email: '' });
    
    // Or reset with form.reset()
    // e.target.reset();
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Name"
      />
      
      <input
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Common Patterns

### Pattern 1: Success/Error Messages

```jsx
function FormWithMessages() {
  const [status, setStatus] = useState(null); // null | 'success' | 'error'
  const [message, setMessage] = useState('');
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      const response = await fetch('/api/submit', { /* ... */ });
      
      if (response.ok) {
        setStatus('success');
        setMessage('Form submitted successfully!');
        
        // Clear message after 3 seconds
        setTimeout(() => {
          setStatus(null);
          setMessage('');
        }, 3000);
      } else {
        setStatus('error');
        setMessage('Submission failed. Please try again.');
      }
    } catch (error) {
      setStatus('error');
      setMessage('Network error. Please check your connection.');
    }
  };
  
  return (
    <div>
      <form onSubmit={handleSubmit}>
        {/* Form fields */}
        <button type="submit">Submit</button>
      </form>
      
      {status === 'success' && (
        <div className="alert alert-success">{message}</div>
      )}
      
      {status === 'error' && (
        <div className="alert alert-error">{message}</div>
      )}
    </div>
  );
}
```

---

### Pattern 2: Submit Button States

```jsx
function SubmitButtonStates() {
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [isValid, setIsValid] = useState(false);
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      
      <button
        type="submit"
        disabled={isSubmitting || !isValid}
        className={isSubmitting ? 'submitting' : ''}
      >
        {isSubmitting ? (
          <>
            <span className="spinner"></span>
            Submitting...
          </>
        ) : (
          'Submit'
        )}
      </button>
    </form>
  );
}
```

---

## Best Practices

**1. Always call e.preventDefault()**
```jsx
const handleSubmit = (e) => {
  e.preventDefault();  // ✅ Prevent page reload
  // Handle submission
};
```

**2. Show loading state during submission**
```jsx
<button type="submit" disabled={isSubmitting}>
  {isSubmitting ? 'Submitting...' : 'Submit'}
</button>
```

**3. Validate before submitting**
```jsx
const handleSubmit = (e) => {
  e.preventDefault();
  
  const errors = validate();
  if (Object.keys(errors).length > 0) {
    return;  // Don't submit if invalid
  }
  
  // Submit
};
```

**4. Handle errors gracefully**
```jsx
try {
  await submitForm();
} catch (error) {
  setError('Submission failed. Please try again.');
}
```

**5. Disable form during submission**
```jsx
<input disabled={isSubmitting} />
<button disabled={isSubmitting}>Submit</button>
```

**6. Reset form after successful submission**
```jsx
if (response.ok) {
  setFormData(initialState);
  // or e.target.reset();
}
```

**7. Provide feedback to user**
```jsx
setStatus('success');
setMessage('Form submitted successfully!');
```

---

**Interview Tips:**
- Use **onSubmit** on form element
- Call **e.preventDefault()** to prevent page reload
- **type="submit"** button triggers submission
- Access form data from **state** (controlled)
- Or use **FormData API** (uncontrolled)
- **Validate before submitting**
- Show **loading state** during submission
- **Disable submit button** while submitting
- Handle **async operations** with try/catch
- **Reset form** after successful submission
- Show **success/error messages**
- Prevent **double submission**
- **File uploads** require FormData
- Don't set **Content-Type** for FormData
- Use **JSON.stringify()** for JSON data
- Set **Content-Type: application/json** for JSON
- **React Hook Form**: handleSubmit wraps your function
- **Formik**: onSubmit receives values and actions
- Submit on **Enter key** (form default behavior)
- Prevent submit in **textarea** with Shift+Enter
- **Server-side validation** is essential
- Store **submission status** in state
- **Clear errors** on new submission
- Use **isSubmitting** state for UI feedback
- **Redirect** after successful submission
- **Token authentication** in headers
- Handle **network errors**
- **Retry logic** for failed submissions
- **Optimistic updates** for better UX
- **Form reset** methods: setState, form.reset(), library reset()

</details>

---

### 80. What is the difference between onChange and onBlur?

<details>
<summary>View Answer</summary>

**onChange vs onBlur**

Both are event handlers for form inputs, but they trigger at different times and serve different purposes.

---

## onChange

**Definition:** Fires every time the input value changes (on every keystroke).

**When it triggers:**
- Every keystroke in text input
- Every character added/deleted
- Select option change
- Checkbox/radio toggle
- Immediately as user types

### Basic Example

```jsx
import React, { useState } from 'react';

function OnChangeExample() {
  const [value, setValue] = useState('');
  const [changeCount, setChangeCount] = useState(0);
  
  const handleChange = (e) => {
    console.log('onChange fired:', e.target.value);
    setValue(e.target.value);
    setChangeCount(prev => prev + 1);
  };
  
  return (
    <div>
      <input
        value={value}
        onChange={handleChange}
        placeholder="Type something..."
      />
      <p>Value: {value}</p>
      <p>onChange called: {changeCount} times</p>
    </div>
  );
}

export default OnChangeExample;
```

**Output when typing "hello":**
- onChange called 5 times
- Fires on: h, e, l, l, o

---

### onChange Use Cases

**1. Real-time search/filter**
```jsx
function SearchBox() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    
    // Filter results as user types
    const filtered = data.filter(item =>
      item.name.toLowerCase().includes(value.toLowerCase())
    );
    setResults(filtered);
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} placeholder="Search..." />
      <ul>
        {results.map(item => <li key={item.id}>{item.name}</li>)}
      </ul>
    </div>
  );
}
```

**2. Character counter**
```jsx
function CharacterCounter() {
  const [text, setText] = useState('');
  const maxLength = 280;
  
  return (
    <div>
      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
        maxLength={maxLength}
      />
      <p>{text.length} / {maxLength} characters</p>
    </div>
  );
}
```

**3. Password strength**
```jsx
function PasswordStrength() {
  const [password, setPassword] = useState('');
  const [strength, setStrength] = useState('');
  
  const calculateStrength = (pwd) => {
    if (pwd.length < 6) return 'Weak';
    if (pwd.length < 10) return 'Medium';
    if (/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/.test(pwd)) return 'Strong';
    return 'Medium';
  };
  
  const handleChange = (e) => {
    const value = e.target.value;
    setPassword(value);
    setStrength(calculateStrength(value));
  };
  
  return (
    <div>
      <input type="password" value={password} onChange={handleChange} />
      <p>Strength: <strong>{strength}</strong></p>
    </div>
  );
}
```

**4. Format input as user types**
```jsx
function PhoneInput() {
  const [phone, setPhone] = useState('');
  
  const formatPhone = (value) => {
    const cleaned = value.replace(/\D/g, '');
    const match = cleaned.match(/^(\d{0,3})(\d{0,3})(\d{0,4})$/);
    
    if (!match) return value;
    
    if (match[3]) return `(${match[1]}) ${match[2]}-${match[3]}`;
    if (match[2]) return `(${match[1]}) ${match[2]}`;
    if (match[1]) return `(${match[1]}`;
    return '';
  };
  
  const handleChange = (e) => {
    const formatted = formatPhone(e.target.value);
    setPhone(formatted);
  };
  
  return (
    <input
      value={phone}
      onChange={handleChange}
      placeholder="(555) 123-4567"
    />
  );
}
```

---

## onBlur

**Definition:** Fires when the input loses focus (user clicks away or tabs out).

**When it triggers:**
- User clicks outside the input
- User presses Tab to next field
- User submits the form
- Only once when leaving the field

### Basic Example

```jsx
import React, { useState } from 'react';

function OnBlurExample() {
  const [value, setValue] = useState('');
  const [blurCount, setBlurCount] = useState(0);
  const [lastBlurValue, setLastBlurValue] = useState('');
  
  const handleBlur = (e) => {
    console.log('onBlur fired:', e.target.value);
    setBlurCount(prev => prev + 1);
    setLastBlurValue(e.target.value);
  };
  
  return (
    <div>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        onBlur={handleBlur}
        placeholder="Type and click away..."
      />
      <p>Current value: {value}</p>
      <p>onBlur called: {blurCount} times</p>
      <p>Last blur value: {lastBlurValue}</p>
    </div>
  );
}

export default OnBlurExample;
```

**Output when typing "hello" and clicking away:**
- onBlur called 1 time
- Only fires when focus is lost

---

### onBlur Use Cases

**1. Validation on blur (better UX)**
```jsx
function EmailInput() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');
  const [touched, setTouched] = useState(false);
  
  const validateEmail = (value) => {
    if (!value) return 'Email is required';
    if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(value)) {
      return 'Invalid email address';
    }
    return '';
  };
  
  const handleBlur = () => {
    setTouched(true);
    const validationError = validateEmail(email);
    setError(validationError);
  };
  
  return (
    <div>
      <input
        value={email}
        onChange={(e) => {
          setEmail(e.target.value);
          // Clear error as user types (after first blur)
          if (touched) {
            setError(validateEmail(e.target.value));
          }
        }}
        onBlur={handleBlur}
        placeholder="Email"
        className={touched && error ? 'invalid' : ''}
      />
      {touched && error && <span className="error">{error}</span>}
    </div>
  );
}
```

**2. Save draft on blur**
```jsx
function AutoSave() {
  const [content, setContent] = useState('');
  const [lastSaved, setLastSaved] = useState(null);
  
  const handleBlur = async () => {
    if (content) {
      await fetch('/api/save-draft', {
        method: 'POST',
        body: JSON.stringify({ content })
      });
      setLastSaved(new Date());
    }
  };
  
  return (
    <div>
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        onBlur={handleBlur}
        placeholder="Start typing..."
      />
      {lastSaved && <p>Last saved: {lastSaved.toLocaleTimeString()}</p>}
    </div>
  );
}
```

**3. Track which fields were touched**
```jsx
function FormWithTouched() {
  const [formData, setFormData] = useState({
    name: '',
    email: ''
  });
  const [touched, setTouched] = useState({
    name: false,
    email: false
  });
  
  const handleBlur = (field) => {
    setTouched(prev => ({ ...prev, [field]: true }));
  };
  
  return (
    <form>
      <input
        name="name"
        value={formData.name}
        onChange={(e) => setFormData({ ...formData, name: e.target.value })}
        onBlur={() => handleBlur('name')}
      />
      {touched.name && !formData.name && <span>Name is required</span>}
      
      <input
        name="email"
        value={formData.email}
        onChange={(e) => setFormData({ ...formData, email: e.target.value })}
        onBlur={() => handleBlur('email')}
      />
      {touched.email && !formData.email && <span>Email is required</span>}
    </form>
  );
}
```

**4. Async validation on blur**
```jsx
function UsernameInput() {
  const [username, setUsername] = useState('');
  const [checking, setChecking] = useState(false);
  const [available, setAvailable] = useState(null);
  
  const handleBlur = async () => {
    if (!username) return;
    
    setChecking(true);
    
    try {
      const response = await fetch(`/api/check-username?username=${username}`);
      const data = await response.json();
      setAvailable(data.available);
    } catch (error) {
      console.error(error);
    } finally {
      setChecking(false);
    }
  };
  
  return (
    <div>
      <input
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        onBlur={handleBlur}
        placeholder="Username"
      />
      {checking && <span>Checking availability...</span>}
      {available === true && <span className="success">✓ Available</span>}
      {available === false && <span className="error">✗ Already taken</span>}
    </div>
  );
}
```

---

## Comparison Table

| Feature | onChange | onBlur |
|---------|----------|--------|
| **Trigger** | Every keystroke | When focus is lost |
| **Frequency** | Multiple times | Once per focus/blur cycle |
| **Performance** | More re-renders | Fewer re-renders |
| **Use For** | Real-time features | Validation, saving |
| **UX** | Immediate feedback | Less intrusive |
| **Validation** | Can be annoying | Better user experience |
| **API Calls** | Needs debouncing | Can call directly |
| **Best For** | Search, formatting | Validation, drafts |

---

## Combined Usage

**Best practice: Use both together**

```jsx
import React, { useState } from 'react';

function CombinedExample() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');
  const [touched, setTouched] = useState(false);
  
  const validateEmail = (value) => {
    if (!value) return 'Email is required';
    if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(value)) {
      return 'Invalid email address';
    }
    return '';
  };
  
  const handleChange = (e) => {
    const value = e.target.value;
    setEmail(value);
    
    // Only validate after field has been touched
    if (touched) {
      setError(validateEmail(value));
    }
  };
  
  const handleBlur = () => {
    setTouched(true);
    setError(validateEmail(email));
  };
  
  return (
    <div>
      <input
        value={email}
        onChange={handleChange}  // Update value, validate if touched
        onBlur={handleBlur}      // Mark as touched, validate
        placeholder="Email"
        className={touched && error ? 'invalid' : ''}
      />
      {touched && error && <span className="error">{error}</span>}
    </div>
  );
}

export default CombinedExample;
```

**Why this pattern is better:**
- ✅ Don't show errors before user interacts (onBlur)
- ✅ Show errors immediately after first blur (onChange)
- ✅ Clear errors as user fixes them (onChange)
- ✅ Better user experience

---

## Validation Strategies

### Strategy 1: Validate on Submit Only

```jsx
// Simplest - validate when form is submitted
const handleSubmit = (e) => {
  e.preventDefault();
  const errors = validate(formData);
  if (Object.keys(errors).length === 0) {
    // Submit
  }
};
```

**Pros:** Simple, no unnecessary validation  
**Cons:** User sees errors late

---

### Strategy 2: Validate on Change

```jsx
// Real-time validation on every keystroke
const handleChange = (e) => {
  setValue(e.target.value);
  setError(validate(e.target.value));
};
```

**Pros:** Immediate feedback  
**Cons:** Can be annoying, shows errors too early

---

### Strategy 3: Validate on Blur

```jsx
// Validate when user leaves field
const handleBlur = () => {
  setTouched(true);
  setError(validate(value));
};
```

**Pros:** Better UX, not intrusive  
**Cons:** Delayed feedback

---

### Strategy 4: Hybrid (Recommended)

```jsx
// Validate on blur first, then on change
const handleChange = (e) => {
  setValue(e.target.value);
  if (touched) {
    setError(validate(e.target.value));
  }
};

const handleBlur = () => {
  setTouched(true);
  setError(validate(value));
};
```

**Pros:** Best of both worlds  
**Cons:** Slightly more complex

---

## With Form Libraries

### React Hook Form

```jsx
import { useForm } from 'react-hook-form';

function RHFExample() {
  const { register, formState: { errors } } = useForm({
    mode: 'onBlur'  // Validate on blur
    // mode: 'onChange'  // Validate on change
    // mode: 'onSubmit'  // Validate on submit (default)
    // mode: 'onTouched'  // Validate on blur, then on change
  });
  
  return (
    <form>
      <input
        {...register('email', {
          required: 'Email is required',
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: 'Invalid email'
          }
        })}
      />
      {errors.email && <span>{errors.email.message}</span>}
    </form>
  );
}
```

---

### Formik

```jsx
import { Formik, Form, Field } from 'formik';

function FormikExample() {
  return (
    <Formik
      initialValues={{ email: '' }}
      validateOnChange={true}  // Validate on change
      validateOnBlur={true}    // Validate on blur
      onSubmit={(values) => console.log(values)}
    >
      {({ errors, touched }) => (
        <Form>
          <Field name="email" />
          {touched.email && errors.email && <span>{errors.email}</span>}
        </Form>
      )}
    </Formik>
  );
}
```

---

## Performance Considerations

### Debounce onChange for Expensive Operations

```jsx
import React, { useState, useEffect } from 'react';

function DebouncedInput() {
  const [value, setValue] = useState('');
  const [debouncedValue, setDebouncedValue] = useState('');
  
  // Debounce the value
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, 500);
    
    return () => clearTimeout(timer);
  }, [value]);
  
  // Use debounced value for expensive operations
  useEffect(() => {
    if (debouncedValue) {
      console.log('API call with:', debouncedValue);
      // Expensive API call here
    }
  }, [debouncedValue]);
  
  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

---

## Best Practices

**1. Use onChange for real-time features**
```jsx
// ✅ Search, filtering, character count
<input onChange={(e) => setQuery(e.target.value)} />
```

**2. Use onBlur for validation**
```jsx
// ✅ Better UX - validate when user leaves field
<input onBlur={() => validate()} />
```

**3. Combine both for best UX**
```jsx
// ✅ Validate on blur, then on change
<input
  onChange={handleChange}
  onBlur={handleBlur}
/>
```

**4. Track touched state**
```jsx
// ✅ Only show errors after user has interacted
{touched && error && <span>{error}</span>}
```

**5. Debounce expensive onChange operations**
```jsx
// ✅ Debounce API calls
useEffect(() => {
  const timer = setTimeout(() => {
    apiCall(value);
  }, 500);
  return () => clearTimeout(timer);
}, [value]);
```

---

**Interview Tips:**
- **onChange** = fires on every keystroke
- **onBlur** = fires when input loses focus
- onChange = **multiple times** per edit
- onBlur = **once** when leaving field
- onChange for **real-time** features (search, counter)
- onBlur for **validation** (better UX)
- onBlur for **API calls** (check availability)
- Combine both for **optimal UX**
- **Touched state** = field has been blurred
- Don't show errors **before** user interacts
- Validate on **blur first**, then on **change**
- **Debounce** onChange for expensive operations
- **React Hook Form** validation modes: onBlur, onChange, onSubmit, onTouched
- **Formik**: validateOnChange, validateOnBlur props
- onBlur = better **performance** (fewer re-renders)
- onChange = better for **formatting** (phone, credit card)
- **File inputs** only have onChange
- **Select** has onChange on option change
- **Checkbox** onChange on check/uncheck
- **onFocus** = when input gains focus
- **touched** pattern improves UX
- **mode: 'onTouched'** = validate on blur, then change (RHF)
- onChange **callback** receives event
- Use **e.target.value** to get current value
- **e.target.name** for multiple inputs
- Don't validate **too early** (annoying)
- Don't validate **too late** (frustrating)
- **Hybrid approach** = best practice
- Clear errors **as user fixes** them
- Save drafts on **onBlur**
- **Async validation** better on blur
- Format input on **onChange**
- Show password strength on **onChange**
- Track analytics on **onBlur**

</details>
