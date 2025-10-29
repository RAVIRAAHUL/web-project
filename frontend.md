// ===== FRONTEND STRUCTURE FOR SUBSCRIPTION MANAGEMENT APP =====
// (React + Redux + React Query + Router DOM)

// =============================
// src/index.js
// =============================
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import { Provider } from 'react-redux';
import store from './store';
import { QueryClient, QueryClientProvider } from 'react-query';
import { BrowserRouter } from 'react-router-dom';

const root = ReactDOM.createRoot(document.getElementById('root'));
const queryClient = new QueryClient();

root.render(
  <React.StrictMode>
    <Provider store={store}>
      <QueryClientProvider client={queryClient}>
        <BrowserRouter>
          <App />
        </BrowserRouter>
      </QueryClientProvider>
    </Provider>
  </React.StrictMode>
);

reportWebVitals();


// =============================
// src/App.js
// =============================
import React from 'react';
import { Routes, Route } from 'react-router-dom';
import Login from './Components/Login';
import Register from './Components/Register';
import ViewSubscription from './User/ViewSubscription';
import DisplaySubscription from './Admin/DisplaySubscription';
import CreateSubscription from './Admin/CreateSubscription';
import ErrorPage from './Components/ErrorPage';

function App() {
  return (
    <div>
      <Routes>
        <Route path="/" element={<Login />} />
        <Route path="/register" element={<Register />} />
        <Route path="/user/view-subscriptions" element={<ViewSubscription />} />
        <Route path="/admin/display-subscriptions" element={<DisplaySubscription />} />
        <Route path="/admin/create-subscription" element={<CreateSubscription />} />
        <Route path="*" element={<ErrorPage />} />
      </Routes>
    </div>
  );
}

export default App;


// =============================
// src/apiconfig.js
// =============================
const API_BASE_URL = 'http://localhost:5000/api';

export const endpoints = {
  login: `${API_BASE_URL}/users/login`,
  register: `${API_BASE_URL}/users/register`,
  getSubscriptions: `${API_BASE_URL}/subscriptions`,
  createSubscription: `${API_BASE_URL}/subscriptions/create`,
  updateSubscription: `${API_BASE_URL}/subscriptions/update`,
  deleteSubscription: `${API_BASE_URL}/subscriptions/delete`,
};

export default API_BASE_URL;


// =============================
// src/Components/Login.jsx
// =============================
import React, { useState } from 'react';
import axios from 'axios';
import { useNavigate } from 'react-router-dom';
import { endpoints } from '../apiconfig';
import './Login.css';

function Login() {
  const navigate = useNavigate();
  const [form, setForm] = useState({ email: '', password: '' });
  const [error, setError] = useState({});

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const validate = () => {
    let errors = {};
    if (!form.email) errors.email = 'Email is required';
    if (!form.password) errors.password = 'Password is required';
    setError(errors);
    return Object.keys(errors).length === 0;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!validate()) return;

    try {
      const res = await axios.post(endpoints.login, form);
      if (res.data.role === 'admin') navigate('/admin/display-subscriptions');
      else navigate('/user/view-subscriptions');
    } catch (err) {
      setError({ general: 'Invalid credentials' });
    }
  };

  return (
    <div className="login-container">
      <h2>Login</h2>
      <form onSubmit={handleSubmit}>
        <input type="email" name="email" placeholder="Email" value={form.email} onChange={handleChange} />
        {error.email && <p className="error">{error.email}</p>}

        <input type="password" name="password" placeholder="Password" value={form.password} onChange={handleChange} />
        {error.password && <p className="error">{error.password}</p>}

        {error.general && <p className="error">{error.general}</p>}

        <button type="submit">Login</button>
      </form>
      <p>Don’t have an account? <span onClick={() => navigate('/register')}>Register</span></p>
    </div>
  );
}

export default Login;


// =============================
// src/Components/Register.jsx
// =============================
import React, { useState } from 'react';
import axios from 'axios';
import { useNavigate } from 'react-router-dom';
import { endpoints } from '../apiconfig';
import './Register.css';

function Register() {
  const navigate = useNavigate();
  const [form, setForm] = useState({ firstName: '', lastName: '', email: '', mobile: '', password: '', confirmPassword: '' });
  const [error, setError] = useState({});

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const validate = () => {
    let errors = {};
    if (!form.firstName) errors.firstName = 'First Name is required';
    if (!form.lastName) errors.lastName = 'Last Name is required';
    if (!form.email) errors.email = 'Email is required';
    if (!form.mobile) errors.mobile = 'Mobile Number is required';
    if (!form.password) errors.password = 'Password is required';
    if (!form.confirmPassword) errors.confirmPassword = 'Confirm Password is required';
    else if (form.password !== form.confirmPassword) errors.confirmPassword = 'Passwords do not match';

    setError(errors);
    return Object.keys(errors).length === 0;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!validate()) return;

    try {
      await axios.post(endpoints.register, form);
      navigate('/');
    } catch (err) {
      setError({ general: 'Registration failed' });
    }
  };

  return (
    <div className="register-container">
      <h2>Create Account</h2>
      <form onSubmit={handleSubmit}>
        <input name="firstName" placeholder="First Name" onChange={handleChange} />
        {error.firstName && <p className="error">{error.firstName}</p>}

        <input name="lastName" placeholder="Last Name" onChange={handleChange} />
        {error.lastName && <p className="error">{error.lastName}</p>}

        <input name="email" placeholder="Email" onChange={handleChange} />
        {error.email && <p className="error">{error.email}</p>}

        <input name="mobile" placeholder="Mobile Number" onChange={handleChange} />
        {error.mobile && <p className="error">{error.mobile}</p>}

        <input type="password" name="password" placeholder="Password" onChange={handleChange} />
        {error.password && <p className="error">{error.password}</p>}

        <input type="password" name="confirmPassword" placeholder="Confirm Password" onChange={handleChange} />
        {error.confirmPassword && <p className="error">{error.confirmPassword}</p>}

        {error.general && <p className="error">{error.general}</p>}

        <button type="submit">Register</button>
      </form>
    </div>
  );
}

export default Register;


// =============================
// src/Components/ErrorPage.jsx
// =============================
import React from 'react';
import './ErrorPage.css';

function ErrorPage() {
  return (
    <div className="error-page">
      <h2>Something Went Wrong</h2>
      <p>We're sorry, but an error occurred. Please try again later.</p>
    </div>
  );
}

export default ErrorPage;


// =============================
// src/User/ViewSubscription.jsx
// =============================
import React from 'react';
import './ViewSubscription.css';

function ViewSubscription() {
  const subscriptions = [];

  return (
    <div className="view-subscription">
      <h2>Your Subscriptions</h2>
      <button>Logout</button>

      {subscriptions.length === 0 ? (
        <p>No subscriptions available.</p>
      ) : (
        <ul>
          {subscriptions.map((sub) => (
            <li key={sub.id}>{sub.serviceName} - ₹{sub.amount}</li>
          ))}
        </ul>
      )}
    </div>
  );
}

export default ViewSubscription;


// =============================
// src/Admin/DisplaySubscription.jsx
// =============================
import React from 'react';

function DisplaySubscription() {
  return (
    <div className="display-subscription">
      <h2>Manage Subscriptions</h2>
      <div className="controls">
        <button>Add Subscription</button>
        <button>Logout</button>
      </div>
      <input type="text" placeholder="Search service..." />
    </div>
  );
}

export default DisplaySubscription;


// =============================
// src/Admin/CreateSubscription.jsx
// =============================
import React, { useState } from 'react';
import './CreateSubscription.css';

function CreateSubscription() {
  const [form, setForm] = useState({
    serviceName: '',
    amount: '',
    billingCycle: '',
    nextRenewalDate: ''
  });

  const [error, setError] = useState({});

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const validate = () => {
    const errors = {};
    if (!form.serviceName) errors.serviceName = 'Service Name is required';
    if (!form.amount || isNaN(form.amount)) errors.amount = 'Valid amount is required';
    if (!form.billingCycle) errors.billingCycle = 'Billing cycle is required';
    if (!form.nextRenewalDate) errors.nextRenewalDate = 'Next renewal date is required';
    setError(errors);
    return Object.keys(errors).length === 0;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!validate()) return;
  };

  return (
    <div className="create-subscription">
      <h2>Add Subscription</h2>
      <form onSubmit={handleSubmit}>
        <input name="serviceName" placeholder="Service Name" onChange={handleChange} />
        {error.serviceName && <p className="error">{error.serviceName}</p>}

        <input name="amount" placeholder="Amount" onChange={handleChange} />
        {error.amount && <p className="error">{error.amount}</p>}

        <input name="billingCycle" placeholder="Billing Cycle" onChange={handleChange} />
        {error.billingCycle && <p className="error">{error.billingCycle}</p>}

        <input type="date" name="nextRenewalDate" onChange={handleChange} />
        {error.nextRenewalDate && <p className="error">{error.nextRenewalDate}</p>}

        <button type="submit">Add Subscription</button>
      </form>
      <button>Back</button>
    </div>
  );
}

export default CreateSubscription;
