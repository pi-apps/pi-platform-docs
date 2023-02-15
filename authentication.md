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
