# Authentication

This is a simple walkthrough on handling user authentication using Pi SDK and API endpoint. For a detailed explanation, please refer to [Pi.authenticate](./SDK_reference.md#authentication) for SDK reference and [/me](./platform_API.md#authentication) for API reference.
## Guide

Here is a list of steps you can follow when you authenticate users for your app:
### 1. Call `authenticate()` of Pi SDK

Using the Pi SDK `authenticate()`, you can obtain user information along with the access token.
```javascript
const authRes = await window.Pi.authenticate(scopes, onIncompletePaymentFound);
```

### 2. Make a GET request to `/me` Pi API endpoint using the access token for verification
To verify the data you got in step 1, you need to make a GET request to `/me` Pi API endpoint, with the access token included in the header. If the access token is valid, it will return a response with [UserDTO](./platform_API.md#UserDTO). However, if the token is invalid, it will return HTTP 401 Unauthorized code.
```javascript
const me = await axios.get('https://api.minepi.com/v2/me', {headers: {'Authorization': `Bearer ${accessToken}}});
```

## Handling User Logout

To handle user logout using the Pi SDK, you can use the following steps:

### 1. Clear User Data

Clear any user data stored in your application, such as user information and access tokens.

```javascript
function clearUserData() {
  // Clear user data from your application
  localStorage.removeItem('user');
  localStorage.removeItem('accessToken');
}
```

### 2. Redirect to Login Page

Redirect the user to the login page or any other appropriate page after logout.

```javascript
function redirectToLogin() {
  window.location.href = '/login';
}
```

### 3. Logout Function

Create a logout function that combines the above steps.

```javascript
function logout() {
  clearUserData();
  redirectToLogin();
}
```

## Handling Token Expiration and Refresh

To handle token expiration and refresh, you can use the following steps:

### 1. Check Token Expiration

Check if the access token is expired before making any API requests.

```javascript
function isTokenExpired(token) {
  const payload = JSON.parse(atob(token.split('.')[1]));
  const expiration = payload.exp;
  const now = Math.floor(Date.now() / 1000);
  return now >= expiration;
}
```

### 2. Refresh Token

If the token is expired, refresh it using the refresh token.

```javascript
async function refreshToken() {
  const refreshToken = localStorage.getItem('refreshToken');
  const response = await axios.post('https://api.minepi.com/v2/refresh', { refreshToken });
  const { accessToken } = response.data;
  localStorage.setItem('accessToken', accessToken);
  return accessToken;
}
```

### 3. API Request with Token Refresh

Create a function to make API requests with automatic token refresh.

```javascript
async function apiRequest(url, options = {}) {
  let accessToken = localStorage.getItem('accessToken');

  if (isTokenExpired(accessToken)) {
    accessToken = await refreshToken();
  }

  const response = await axios.get(url, {
    ...options,
    headers: {
      ...options.headers,
      'Authorization': `Bearer ${accessToken}`,
    },
  });

  return response.data;
}
```
