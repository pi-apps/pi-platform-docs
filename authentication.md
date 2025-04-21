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

## Integrating Pi SDK with Third-Party Authentication Services

Integrating Pi SDK with third-party authentication services like OAuth or OpenID Connect can provide a seamless login experience for users. Here are the steps to integrate Pi SDK with third-party authentication services:

### 1. Use Pi SDK's `authenticate` method to obtain user information and access tokens

```javascript
const authRes = await window.Pi.authenticate(scopes, onIncompletePaymentFound);
const accessToken = authRes.accessToken;
```

### 2. Integrate with third-party authentication services

Use the access token obtained from the Pi SDK to verify user identity and permissions with the third-party service. For example, using OAuth:

```javascript
const oauthResponse = await axios.post('https://third-party-auth-service.com/oauth/token', {
  grant_type: 'authorization_code',
  code: accessToken,
  redirect_uri: 'https://your-app.com/callback',
  client_id: 'YOUR_CLIENT_ID',
  client_secret: 'YOUR_CLIENT_SECRET'
});
const thirdPartyAccessToken = oauthResponse.data.access_token;
```

### 3. Verify user identity and permissions

Use the third-party access token to verify user identity and permissions with the third-party service:

```javascript
const userInfo = await axios.get('https://third-party-auth-service.com/userinfo', {
  headers: {
    'Authorization': `Bearer ${thirdPartyAccessToken}`
  }
});
```

### Benefits of Integrating with Third-Party Authentication Services

* Provides a seamless login experience for users
* Enhances security by leveraging third-party authentication mechanisms
* Simplifies user management by using existing authentication services
