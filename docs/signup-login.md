# Frontend Integration Guide - ApplyPal Backend API

This guide explains how to integrate your frontend application with the ApplyPal backend API.

## Base URL

- **Development**: `http://localhost:3000`
- **Production**: `https://applypal-backend.onrender.com`

## API Documentation

- **Swagger UI**: `{BASE_URL}/api/docs`
- **Interactive Testing**: Available at the Swagger endpoint

## Authentication

The API uses JWT (JSON Web Token) for authentication. Include the token in the Authorization header for protected endpoints.

```
Authorization: Bearer <your-jwt-token>
```

## API Endpoints

### Ambassador Endpoints

#### Register Ambassador
```
POST /ambassador/signup
Content-Type: application/json

{
  "fullName": "John Doe",
  "email": "john.doe@example.com",
  "university": "Harvard University",
  "password": "SecurePassword123!",
  "confirmPassword": "SecurePassword123!",
  "role": "ambassador"
}
```

**Response:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "abc123",
    "fullName": "John Doe",
    "email": "john.doe@example.com",
    "university": "Harvard University",
    "role": "ambassador"
  }
}
```

#### Login Ambassador
```
POST /ambassador/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "SecurePassword123!",
  "role": "ambassador"
}
```

**Response:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "abc123",
    "fullName": "John Doe",
    "email": "john.doe@example.com",
    "university": "Harvard University",
    "role": "ambassador"
  }
}
```

#### Get Ambassador Profile (Protected)
```
GET /ambassador/profile
Authorization: Bearer <jwt-token>
```

**Response:**
```json
{
  "id": "abc123",
  "fullName": "John Doe",
  "email": "john.doe@example.com",
  "university": "Harvard University",
  "role": "ambassador",
  "createdAt": "2025-01-28T10:30:00.000Z",
  "updatedAt": "2025-01-28T10:30:00.000Z"
}
```

### University Endpoints

#### Register University
```
POST /university/signup
Content-Type: application/json

{
  "fullName": "MIT Admissions Office",
  "email": "admissions@mit.edu",
  "university": "Massachusetts Institute of Technology",
  "password": "SecurePassword123!",
  "confirmPassword": "SecurePassword123!",
  "role": "university"
}
```

**Response:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "def456",
    "fullName": "MIT Admissions Office",
    "email": "admissions@mit.edu",
    "university": "Massachusetts Institute of Technology",
    "role": "university"
  }
}
```

#### Login University
```
POST /university/login
Content-Type: application/json

{
  "email": "admissions@mit.edu",
  "password": "SecurePassword123!",
  "role": "university"
}
```

**Response:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "def456",
    "fullName": "MIT Admissions Office",
    "email": "admissions@mit.edu",
    "university": "Massachusetts Institute of Technology",
    "role": "university"
  }
}
```

#### Get University Profile (Protected)
```
GET /university/profile
Authorization: Bearer <jwt-token>
```

**Response:**
```json
{
  "id": "def456",
  "fullName": "MIT Admissions Office",
  "email": "admissions@mit.edu",
  "university": "Massachusetts Institute of Technology",
  "role": "university",
  "createdAt": "2025-01-28T10:30:00.000Z",
  "updatedAt": "2025-01-28T10:30:00.000Z"
}
```

## Frontend Implementation Examples



```javascript
// Base API configuration
const API_BASE_URL = 'http://localhost:3000'; // or your production URL

// Helper function for API calls
async function apiCall(endpoint, options = {}) {
  const url = `${API_BASE_URL}${endpoint}`;
  const config = {
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    },
    ...options,
  };

  try {
    const response = await fetch(url, config);
    const data = await response.json();
    
    if (!response.ok) {
      throw new Error(data.message || 'API call failed');
    }
    
    return data;
  } catch (error) {
    console.error('API Error:', error);
    throw error;
  }
}

// Ambassador signup
async function signupAmbassador(userData) {
  return apiCall('/ambassador/signup', {
    method: 'POST',
    body: JSON.stringify(userData),
  });
}

// Ambassador login
async function loginAmbassador(credentials) {
  return apiCall('/ambassador/login', {
    method: 'POST',
    body: JSON.stringify(credentials),
  });
}

// Get ambassador profile (requires token)
async function getAmbassadorProfile(token) {
  return apiCall('/ambassador/profile', {
    headers: {
      'Authorization': `Bearer ${token}`,
    },
  });
}

// University signup
async function signupUniversity(userData) {
  return apiCall('/university/signup', {
    method: 'POST',
    body: JSON.stringify(userData),
  });
}

// University login
async function loginUniversity(credentials) {
  return apiCall('/university/login', {
    method: 'POST',
    body: JSON.stringify(credentials),
  });
}

// Get university profile (requires token)
async function getUniversityProfile(token) {
  return apiCall('/university/profile', {
    headers: {
      'Authorization': `Bearer ${token}`,
    },
  });
}
```

### React Hook Example

```javascript
import { useState, useEffect } from 'react';

// Custom hook for API calls
function useApi() {
  const [token, setToken] = useState(localStorage.getItem('token'));
  const [user, setUser] = useState(null);

  // Store token in localStorage
  const saveToken = (newToken) => {
    setToken(newToken);
    localStorage.setItem('token', newToken);
  };

  // Remove token from localStorage
  const logout = () => {
    setToken(null);
    setUser(null);
    localStorage.removeItem('token');
  };

  // API call with authentication
  const apiCall = async (endpoint, options = {}) => {
    const url = `http://localhost:3000${endpoint}`;
    const config = {
      headers: {
        'Content-Type': 'application/json',
        ...(token && { 'Authorization': `Bearer ${token}` }),
        ...options.headers,
      },
      ...options,
    };

    const response = await fetch(url, config);
    const data = await response.json();
    
    if (!response.ok) {
      throw new Error(data.message || 'API call failed');
    }
    
    return data;
  };

  return {
    token,
    user,
    setUser,
    saveToken,
    logout,
    apiCall,
  };
}

// Usage in component
function LoginComponent() {
  const { saveToken, setUser } = useApi();
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    role: 'ambassador',
  });

  const handleLogin = async (e) => {
    e.preventDefault();
    
    try {
      const endpoint = formData.role === 'ambassador' 
        ? '/ambassador/login' 
        : '/university/login';
      
      const response = await apiCall(endpoint, {
        method: 'POST',
        body: JSON.stringify(formData),
      });

      saveToken(response.accessToken);
      setUser(response.user);
      
      // Redirect to dashboard
      window.location.href = '/dashboard';
    } catch (error) {
      alert('Login failed: ' + error.message);
    }
  };

  return (
    <form onSubmit={handleLogin}>
      <input
        type="email"
        placeholder="Email"
        value={formData.email}
        onChange={(e) => setFormData({...formData, email: e.target.value})}
        required
      />
      <input
        type="password"
        placeholder="Password"
        value={formData.password}
        onChange={(e) => setFormData({...formData, password: e.target.value})}
        required
      />
      <select
        value={formData.role}
        onChange={(e) => setFormData({...formData, role: e.target.value})}
      >
        <option value="ambassador">Ambassador</option>
        <option value="university">University</option>
      </select>
      <button type="submit">Login</button>
    </form>
  );
}
```

import axios from 'axios';

// Create axios instance
const api = axios.create({
  baseURL: 'http://localhost:3000',
  headers: {
    'Content-Type': 'application/json',
  },
});

// Add token to requests
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Handle token expiration
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

// API functions
export const ambassadorApi = {
  signup: (data) => api.post('/ambassador/signup', data),
  login: (data) => api.post('/ambassador/login', data),
  getProfile: () => api.get('/ambassador/profile'),
};

export const universityApi = {
  signup: (data) => api.post('/university/signup', data),
  login: (data) => api.post('/university/login', data),
  getProfile: () => api.get('/university/profile'),
};
```

## Error Handling

The API returns standardized error responses:

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "error": "Bad Request"
}
```

Common HTTP status codes:
- `200` - Success
- `201` - Created (for signup)
- `400` - Bad Request (validation errors)
- `401` - Unauthorized (invalid token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found
- `500` - Internal Server Error

## Validation Rules

### Signup Validation
- `fullName`: Required, string, not empty
- `email`: Required, valid email format, unique
- `university`: Required, string, not empty
- `password`: Required, string, minimum length
- `confirmPassword`: Must match password
- `role`: Must be either "ambassador" or "university"

### Login Validation
- `email`: Required, valid email format
- `password`: Required, string
- `role`: Must be either "ambassador" or "university"

## Testing

You can test the API endpoints using:

1. **Swagger UI**: Visit `{BASE_URL}/api/docs`
2. **Postman**: Import the API endpoints
3. **cURL**: Use command line tools
4. **Frontend Application**: Implement the integration

## Security Notes

- Always use HTTPS in production
- Store JWT tokens securely (httpOnly cookies recommended)
- Implement token refresh mechanism
- Validate all user inputs on the frontend
- Handle token expiration gracefully
- Use environment variables for API URLs

## Support

For API-related issues or questions, refer to the Swagger documentation at `{BASE_URL}/api/docs` or contact the backend development team.
