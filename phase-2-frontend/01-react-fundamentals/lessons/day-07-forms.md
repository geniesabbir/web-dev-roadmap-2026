# Day 7: Forms in React

## Controlled Components

In React, form elements maintain their own state. A controlled component is one where React state is the "single source of truth" for input values.

### Basic Controlled Input

```jsx
function ControlledInput() {
    const [value, setValue] = useState('');

    const handleChange = (e) => {
        setValue(e.target.value);
    };

    return (
        <input
            type="text"
            value={value}
            onChange={handleChange}
            placeholder="Type something..."
        />
    );
}
```

### Why Controlled Components?

```jsx
// Benefits:
// 1. Single source of truth
// 2. Instant access to value
// 3. Validate on every keystroke
// 4. Conditionally disable submit
// 5. Format input as user types
// 6. Submit with JavaScript

function ControlledForm() {
    const [email, setEmail] = useState('');
    const [isValid, setIsValid] = useState(false);

    const handleChange = (e) => {
        const value = e.target.value;
        setEmail(value);
        setIsValid(value.includes('@'));  // Instant validation
    };

    const handleSubmit = (e) => {
        e.preventDefault();
        if (isValid) {
            console.log('Submitting:', email);
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                type="email"
                value={email}
                onChange={handleChange}
                style={{ borderColor: isValid ? 'green' : 'red' }}
            />
            <button type="submit" disabled={!isValid}>
                Submit
            </button>
        </form>
    );
}
```

---

## Form Input Types

### Text Inputs

```jsx
function TextInputs() {
    const [formData, setFormData] = useState({
        username: '',
        email: '',
        password: '',
        phone: '',
        website: ''
    });

    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData(prev => ({
            ...prev,
            [name]: value
        }));
    };

    return (
        <form>
            <input
                type="text"
                name="username"
                value={formData.username}
                onChange={handleChange}
                placeholder="Username"
            />

            <input
                type="email"
                name="email"
                value={formData.email}
                onChange={handleChange}
                placeholder="Email"
            />

            <input
                type="password"
                name="password"
                value={formData.password}
                onChange={handleChange}
                placeholder="Password"
            />

            <input
                type="tel"
                name="phone"
                value={formData.phone}
                onChange={handleChange}
                placeholder="Phone"
            />

            <input
                type="url"
                name="website"
                value={formData.website}
                onChange={handleChange}
                placeholder="Website"
            />
        </form>
    );
}
```

### Textarea

```jsx
function TextareaExample() {
    const [bio, setBio] = useState('');
    const maxLength = 500;

    return (
        <div>
            <textarea
                value={bio}
                onChange={(e) => setBio(e.target.value)}
                placeholder="Tell us about yourself..."
                rows={5}
                maxLength={maxLength}
            />
            <p>{bio.length}/{maxLength} characters</p>
        </div>
    );
}
```

### Checkbox

```jsx
function CheckboxExamples() {
    // Single checkbox
    const [isAgreed, setIsAgreed] = useState(false);

    // Multiple checkboxes
    const [preferences, setPreferences] = useState({
        newsletter: false,
        notifications: true,
        marketing: false
    });

    const handlePreferenceChange = (e) => {
        const { name, checked } = e.target;
        setPreferences(prev => ({
            ...prev,
            [name]: checked
        }));
    };

    return (
        <div>
            {/* Single checkbox */}
            <label>
                <input
                    type="checkbox"
                    checked={isAgreed}
                    onChange={(e) => setIsAgreed(e.target.checked)}
                />
                I agree to the terms
            </label>

            {/* Multiple checkboxes */}
            <div>
                <label>
                    <input
                        type="checkbox"
                        name="newsletter"
                        checked={preferences.newsletter}
                        onChange={handlePreferenceChange}
                    />
                    Subscribe to newsletter
                </label>

                <label>
                    <input
                        type="checkbox"
                        name="notifications"
                        checked={preferences.notifications}
                        onChange={handlePreferenceChange}
                    />
                    Enable notifications
                </label>

                <label>
                    <input
                        type="checkbox"
                        name="marketing"
                        checked={preferences.marketing}
                        onChange={handlePreferenceChange}
                    />
                    Receive marketing emails
                </label>
            </div>
        </div>
    );
}
```

### Radio Buttons

```jsx
function RadioExample() {
    const [plan, setPlan] = useState('basic');

    return (
        <div>
            <label>
                <input
                    type="radio"
                    name="plan"
                    value="basic"
                    checked={plan === 'basic'}
                    onChange={(e) => setPlan(e.target.value)}
                />
                Basic - $9/month
            </label>

            <label>
                <input
                    type="radio"
                    name="plan"
                    value="pro"
                    checked={plan === 'pro'}
                    onChange={(e) => setPlan(e.target.value)}
                />
                Pro - $19/month
            </label>

            <label>
                <input
                    type="radio"
                    name="plan"
                    value="enterprise"
                    checked={plan === 'enterprise'}
                    onChange={(e) => setPlan(e.target.value)}
                />
                Enterprise - $49/month
            </label>

            <p>Selected: {plan}</p>
        </div>
    );
}
```

### Select Dropdown

```jsx
function SelectExamples() {
    // Single select
    const [country, setCountry] = useState('');

    // Multiple select
    const [skills, setSkills] = useState([]);

    const handleMultiSelectChange = (e) => {
        const options = e.target.options;
        const selected = [];
        for (let i = 0; i < options.length; i++) {
            if (options[i].selected) {
                selected.push(options[i].value);
            }
        }
        setSkills(selected);
    };

    return (
        <div>
            {/* Single select */}
            <select
                value={country}
                onChange={(e) => setCountry(e.target.value)}
            >
                <option value="">Select a country</option>
                <option value="us">United States</option>
                <option value="uk">United Kingdom</option>
                <option value="ca">Canada</option>
                <option value="au">Australia</option>
            </select>

            {/* Multiple select */}
            <select
                multiple
                value={skills}
                onChange={handleMultiSelectChange}
                style={{ height: '100px' }}
            >
                <option value="javascript">JavaScript</option>
                <option value="react">React</option>
                <option value="node">Node.js</option>
                <option value="python">Python</option>
            </select>
        </div>
    );
}
```

### File Input

```jsx
function FileInputExample() {
    const [selectedFile, setSelectedFile] = useState(null);
    const [preview, setPreview] = useState(null);

    const handleFileChange = (e) => {
        const file = e.target.files[0];
        setSelectedFile(file);

        // Create preview for images
        if (file && file.type.startsWith('image/')) {
            const reader = new FileReader();
            reader.onloadend = () => {
                setPreview(reader.result);
            };
            reader.readAsDataURL(file);
        } else {
            setPreview(null);
        }
    };

    return (
        <div>
            <input
                type="file"
                accept="image/*"
                onChange={handleFileChange}
            />

            {selectedFile && (
                <div>
                    <p>Selected: {selectedFile.name}</p>
                    <p>Size: {(selectedFile.size / 1024).toFixed(2)} KB</p>
                </div>
            )}

            {preview && (
                <img
                    src={preview}
                    alt="Preview"
                    style={{ maxWidth: '200px', marginTop: '10px' }}
                />
            )}
        </div>
    );
}
```

---

## Form Validation

### Basic Validation

```jsx
function BasicValidation() {
    const [formData, setFormData] = useState({
        email: '',
        password: ''
    });
    const [errors, setErrors] = useState({});

    const validate = () => {
        const newErrors = {};

        if (!formData.email) {
            newErrors.email = 'Email is required';
        } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
            newErrors.email = 'Email is invalid';
        }

        if (!formData.password) {
            newErrors.password = 'Password is required';
        } else if (formData.password.length < 8) {
            newErrors.password = 'Password must be at least 8 characters';
        }

        setErrors(newErrors);
        return Object.keys(newErrors).length === 0;
    };

    const handleSubmit = (e) => {
        e.preventDefault();
        if (validate()) {
            console.log('Form submitted:', formData);
        }
    };

    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData(prev => ({ ...prev, [name]: value }));
        // Clear error when user starts typing
        if (errors[name]) {
            setErrors(prev => ({ ...prev, [name]: '' }));
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <div>
                <input
                    type="email"
                    name="email"
                    value={formData.email}
                    onChange={handleChange}
                    placeholder="Email"
                    style={{ borderColor: errors.email ? 'red' : '#ccc' }}
                />
                {errors.email && <span className="error">{errors.email}</span>}
            </div>

            <div>
                <input
                    type="password"
                    name="password"
                    value={formData.password}
                    onChange={handleChange}
                    placeholder="Password"
                    style={{ borderColor: errors.password ? 'red' : '#ccc' }}
                />
                {errors.password && <span className="error">{errors.password}</span>}
            </div>

            <button type="submit">Submit</button>
        </form>
    );
}
```

### Real-time Validation

```jsx
function RealTimeValidation() {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const [touched, setTouched] = useState({ email: false, password: false });

    const errors = {
        email: !email
            ? 'Email is required'
            : !/\S+@\S+\.\S+/.test(email)
            ? 'Invalid email format'
            : '',
        password: !password
            ? 'Password is required'
            : password.length < 8
            ? 'Password must be at least 8 characters'
            : !/[A-Z]/.test(password)
            ? 'Password must contain uppercase letter'
            : !/[0-9]/.test(password)
            ? 'Password must contain a number'
            : ''
    };

    const isValid = !errors.email && !errors.password;

    return (
        <form onSubmit={(e) => e.preventDefault()}>
            <div>
                <input
                    type="email"
                    value={email}
                    onChange={(e) => setEmail(e.target.value)}
                    onBlur={() => setTouched(prev => ({ ...prev, email: true }))}
                    placeholder="Email"
                />
                {touched.email && errors.email && (
                    <span className="error">{errors.email}</span>
                )}
            </div>

            <div>
                <input
                    type="password"
                    value={password}
                    onChange={(e) => setPassword(e.target.value)}
                    onBlur={() => setTouched(prev => ({ ...prev, password: true }))}
                    placeholder="Password"
                />
                {touched.password && errors.password && (
                    <span className="error">{errors.password}</span>
                )}
            </div>

            <button type="submit" disabled={!isValid}>
                Submit
            </button>
        </form>
    );
}
```

---

## Custom Form Hook

```jsx
function useForm(initialValues, validate) {
    const [values, setValues] = useState(initialValues);
    const [errors, setErrors] = useState({});
    const [touched, setTouched] = useState({});
    const [isSubmitting, setIsSubmitting] = useState(false);

    const handleChange = (e) => {
        const { name, value, type, checked } = e.target;
        setValues(prev => ({
            ...prev,
            [name]: type === 'checkbox' ? checked : value
        }));
    };

    const handleBlur = (e) => {
        const { name } = e.target;
        setTouched(prev => ({ ...prev, [name]: true }));

        if (validate) {
            const validationErrors = validate(values);
            setErrors(validationErrors);
        }
    };

    const handleSubmit = (onSubmit) => async (e) => {
        e.preventDefault();

        // Touch all fields
        const allTouched = Object.keys(values).reduce((acc, key) => {
            acc[key] = true;
            return acc;
        }, {});
        setTouched(allTouched);

        // Validate
        if (validate) {
            const validationErrors = validate(values);
            setErrors(validationErrors);

            if (Object.keys(validationErrors).length > 0) {
                return;
            }
        }

        setIsSubmitting(true);
        try {
            await onSubmit(values);
        } finally {
            setIsSubmitting(false);
        }
    };

    const resetForm = () => {
        setValues(initialValues);
        setErrors({});
        setTouched({});
    };

    return {
        values,
        errors,
        touched,
        isSubmitting,
        handleChange,
        handleBlur,
        handleSubmit,
        resetForm,
        setValues
    };
}

// Usage
function SignupForm() {
    const validate = (values) => {
        const errors = {};

        if (!values.email) {
            errors.email = 'Email is required';
        } else if (!/\S+@\S+\.\S+/.test(values.email)) {
            errors.email = 'Invalid email';
        }

        if (!values.password) {
            errors.password = 'Password is required';
        } else if (values.password.length < 8) {
            errors.password = 'Must be 8+ characters';
        }

        return errors;
    };

    const form = useForm(
        { email: '', password: '', remember: false },
        validate
    );

    const onSubmit = async (values) => {
        console.log('Submitted:', values);
        await new Promise(r => setTimeout(r, 1000));
        form.resetForm();
    };

    return (
        <form onSubmit={form.handleSubmit(onSubmit)}>
            <div>
                <input
                    name="email"
                    type="email"
                    value={form.values.email}
                    onChange={form.handleChange}
                    onBlur={form.handleBlur}
                    placeholder="Email"
                />
                {form.touched.email && form.errors.email && (
                    <span className="error">{form.errors.email}</span>
                )}
            </div>

            <div>
                <input
                    name="password"
                    type="password"
                    value={form.values.password}
                    onChange={form.handleChange}
                    onBlur={form.handleBlur}
                    placeholder="Password"
                />
                {form.touched.password && form.errors.password && (
                    <span className="error">{form.errors.password}</span>
                )}
            </div>

            <label>
                <input
                    name="remember"
                    type="checkbox"
                    checked={form.values.remember}
                    onChange={form.handleChange}
                />
                Remember me
            </label>

            <button type="submit" disabled={form.isSubmitting}>
                {form.isSubmitting ? 'Submitting...' : 'Sign Up'}
            </button>
        </form>
    );
}
```

---

## Uncontrolled Components

Use `ref` to access form values directly from the DOM.

```jsx
function UncontrolledForm() {
    const nameRef = useRef(null);
    const emailRef = useRef(null);
    const fileRef = useRef(null);

    const handleSubmit = (e) => {
        e.preventDefault();

        const formData = {
            name: nameRef.current.value,
            email: emailRef.current.value,
            file: fileRef.current.files[0]
        };

        console.log('Submitted:', formData);
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                ref={nameRef}
                type="text"
                defaultValue="Default Name"
                placeholder="Name"
            />

            <input
                ref={emailRef}
                type="email"
                placeholder="Email"
            />

            {/* File inputs are always uncontrolled */}
            <input
                ref={fileRef}
                type="file"
            />

            <button type="submit">Submit</button>
        </form>
    );
}
```

---

## Form Libraries

### React Hook Form (Recommended)

```jsx
import { useForm } from 'react-hook-form';

function ReactHookFormExample() {
    const {
        register,
        handleSubmit,
        formState: { errors, isSubmitting },
        reset
    } = useForm();

    const onSubmit = async (data) => {
        console.log(data);
        await new Promise(r => setTimeout(r, 1000));
        reset();
    };

    return (
        <form onSubmit={handleSubmit(onSubmit)}>
            <input
                {...register('email', {
                    required: 'Email is required',
                    pattern: {
                        value: /\S+@\S+\.\S+/,
                        message: 'Invalid email'
                    }
                })}
                type="email"
                placeholder="Email"
            />
            {errors.email && <span>{errors.email.message}</span>}

            <input
                {...register('password', {
                    required: 'Password is required',
                    minLength: {
                        value: 8,
                        message: 'Must be 8+ characters'
                    }
                })}
                type="password"
                placeholder="Password"
            />
            {errors.password && <span>{errors.password.message}</span>}

            <button type="submit" disabled={isSubmitting}>
                {isSubmitting ? 'Submitting...' : 'Submit'}
            </button>
        </form>
    );
}
```

---

## Practice Exercises

### Exercise: Complete Registration Form

```jsx
// Create a registration form with:
// - Name, email, password, confirm password
// - Terms checkbox
// - Country select
// - Full validation
// - Submit handling

// Your code here
```

**Solution:**
```jsx
function RegistrationForm() {
    const [formData, setFormData] = useState({
        name: '',
        email: '',
        password: '',
        confirmPassword: '',
        country: '',
        acceptTerms: false
    });

    const [errors, setErrors] = useState({});
    const [isSubmitting, setIsSubmitting] = useState(false);
    const [isSuccess, setIsSuccess] = useState(false);

    const countries = [
        { code: 'us', name: 'United States' },
        { code: 'uk', name: 'United Kingdom' },
        { code: 'ca', name: 'Canada' },
        { code: 'au', name: 'Australia' }
    ];

    const validate = () => {
        const newErrors = {};

        if (!formData.name.trim()) {
            newErrors.name = 'Name is required';
        }

        if (!formData.email) {
            newErrors.email = 'Email is required';
        } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
            newErrors.email = 'Invalid email format';
        }

        if (!formData.password) {
            newErrors.password = 'Password is required';
        } else if (formData.password.length < 8) {
            newErrors.password = 'Password must be at least 8 characters';
        } else if (!/[A-Z]/.test(formData.password)) {
            newErrors.password = 'Password must contain an uppercase letter';
        } else if (!/[0-9]/.test(formData.password)) {
            newErrors.password = 'Password must contain a number';
        }

        if (formData.password !== formData.confirmPassword) {
            newErrors.confirmPassword = 'Passwords do not match';
        }

        if (!formData.country) {
            newErrors.country = 'Please select a country';
        }

        if (!formData.acceptTerms) {
            newErrors.acceptTerms = 'You must accept the terms';
        }

        return newErrors;
    };

    const handleChange = (e) => {
        const { name, value, type, checked } = e.target;
        setFormData(prev => ({
            ...prev,
            [name]: type === 'checkbox' ? checked : value
        }));

        // Clear error when field is modified
        if (errors[name]) {
            setErrors(prev => ({ ...prev, [name]: '' }));
        }
    };

    const handleSubmit = async (e) => {
        e.preventDefault();

        const validationErrors = validate();
        setErrors(validationErrors);

        if (Object.keys(validationErrors).length > 0) {
            return;
        }

        setIsSubmitting(true);

        try {
            // Simulate API call
            await new Promise(r => setTimeout(r, 1500));
            console.log('Registration data:', formData);
            setIsSuccess(true);
        } catch (error) {
            console.error('Error:', error);
        } finally {
            setIsSubmitting(false);
        }
    };

    if (isSuccess) {
        return (
            <div style={{ textAlign: 'center', padding: '40px' }}>
                <h2>Registration Successful!</h2>
                <p>Welcome, {formData.name}!</p>
            </div>
        );
    }

    return (
        <form onSubmit={handleSubmit} style={{ maxWidth: '400px', margin: '0 auto' }}>
            <h2>Create Account</h2>

            <div style={{ marginBottom: '15px' }}>
                <input
                    type="text"
                    name="name"
                    value={formData.name}
                    onChange={handleChange}
                    placeholder="Full Name"
                    style={{
                        width: '100%',
                        padding: '10px',
                        borderColor: errors.name ? 'red' : '#ccc'
                    }}
                />
                {errors.name && <span style={{ color: 'red', fontSize: '12px' }}>{errors.name}</span>}
            </div>

            <div style={{ marginBottom: '15px' }}>
                <input
                    type="email"
                    name="email"
                    value={formData.email}
                    onChange={handleChange}
                    placeholder="Email"
                    style={{
                        width: '100%',
                        padding: '10px',
                        borderColor: errors.email ? 'red' : '#ccc'
                    }}
                />
                {errors.email && <span style={{ color: 'red', fontSize: '12px' }}>{errors.email}</span>}
            </div>

            <div style={{ marginBottom: '15px' }}>
                <input
                    type="password"
                    name="password"
                    value={formData.password}
                    onChange={handleChange}
                    placeholder="Password"
                    style={{
                        width: '100%',
                        padding: '10px',
                        borderColor: errors.password ? 'red' : '#ccc'
                    }}
                />
                {errors.password && <span style={{ color: 'red', fontSize: '12px' }}>{errors.password}</span>}
            </div>

            <div style={{ marginBottom: '15px' }}>
                <input
                    type="password"
                    name="confirmPassword"
                    value={formData.confirmPassword}
                    onChange={handleChange}
                    placeholder="Confirm Password"
                    style={{
                        width: '100%',
                        padding: '10px',
                        borderColor: errors.confirmPassword ? 'red' : '#ccc'
                    }}
                />
                {errors.confirmPassword && <span style={{ color: 'red', fontSize: '12px' }}>{errors.confirmPassword}</span>}
            </div>

            <div style={{ marginBottom: '15px' }}>
                <select
                    name="country"
                    value={formData.country}
                    onChange={handleChange}
                    style={{
                        width: '100%',
                        padding: '10px',
                        borderColor: errors.country ? 'red' : '#ccc'
                    }}
                >
                    <option value="">Select Country</option>
                    {countries.map(c => (
                        <option key={c.code} value={c.code}>{c.name}</option>
                    ))}
                </select>
                {errors.country && <span style={{ color: 'red', fontSize: '12px' }}>{errors.country}</span>}
            </div>

            <div style={{ marginBottom: '15px' }}>
                <label style={{ display: 'flex', alignItems: 'center', gap: '8px' }}>
                    <input
                        type="checkbox"
                        name="acceptTerms"
                        checked={formData.acceptTerms}
                        onChange={handleChange}
                    />
                    I accept the terms and conditions
                </label>
                {errors.acceptTerms && <span style={{ color: 'red', fontSize: '12px' }}>{errors.acceptTerms}</span>}
            </div>

            <button
                type="submit"
                disabled={isSubmitting}
                style={{
                    width: '100%',
                    padding: '12px',
                    backgroundColor: '#0066cc',
                    color: 'white',
                    border: 'none',
                    cursor: isSubmitting ? 'not-allowed' : 'pointer',
                    opacity: isSubmitting ? 0.7 : 1
                }}
            >
                {isSubmitting ? 'Creating Account...' : 'Create Account'}
            </button>
        </form>
    );
}
```

---

## Key Takeaways

1. **Controlled components** - React state is source of truth
2. **Use value + onChange** - For controlled inputs
3. **Handle all input types** - Text, checkbox, radio, select, file
4. **Validate on submit and blur** - Show errors appropriately
5. **Use touched state** - Don't show errors before user interacts
6. **Consider form libraries** - React Hook Form for complex forms
7. **Prevent default** - Always on form submit

---

## Self-Check Questions

1. What is a controlled component?
2. How do you handle checkboxes vs text inputs?
3. What's the difference between value and defaultValue?
4. When should you use uncontrolled components?
5. How do you handle file inputs?
6. What is form validation best practice?

---

**Next Lesson:** [Day 8 - useEffect](./day-08-useEffect.md)
