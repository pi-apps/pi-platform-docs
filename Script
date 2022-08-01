# Frontend Javascript SDK

The JS SDK is the frontend SDK, designed to be used in your HTML pages or Single-Page Apps, served in the Pi Browser.

In order to enable the SDK to function correctly, you need to declare your apps on the Developer Portal (open
develop.pi in the Pi Browser to access the Developer Portal).

This SDK is **not** for a server-side NodeJS app.


## Installation

Add the following `script` tags to all pages where you need to call the Pi Apps SDK:

```html
<script src="https://sdk.minepi.com/pi-sdk.js"></script>
<script>Pi.init({ version: "2.0" })</script>
```

This will load the Pi Network JS SDK as a global `window.Pi` object.

## Usage

### Authenticate a user

You cannot perform any user-related operations (e.g read the user's info, request a payment from them) until you
have successfully authenticated the user. The first time, they will be presented with a dialog asking for
their consent to share their data with your app.

```javascript
// Authenticate the user, and get permission to request payments from them:
const scopes = ['payments'];

// Read more about this callback in the SDK reference:
function onIncompletePaymentFound(payment) { /* ... */ };

Pi.authenticate(scopes, onIncompletePaymentFound).then(function(auth) {
  console.log(`Hi there! You're ready to make payments!`);
}).catch(function(error) {
  console.error(error);
});
```

### Request a payment

The `createPayment` method enables you to request a payment from the current user to your app's account.

The user will be prompted with a modal provided by the Pi Wallet, enabling them to sign the
transaction and submit it to the Pi blockchain.

```javascript

Pi.createPayment({
  // Amount of π to be paid:
  amount: 3.14,
  // An explanation of the payment - will be shown to the user:
  memo: "...", // e.g: "Digital kitten #1234",
  // An arbitrary developer-provided metadata object - for your own usage:
  metadata: { /* ... */ }, // e.g: { kittenId: 1234 }
}, {
  // Callbacks you need to implement - read more about those in the detailed docs linked below:
  onReadyForServerApproval: function(paymentId) { /* ... */ },
  onReadyForServerCompletion: function(paymentId, txid) { /* ... */ },
  onCancel: function(paymentId) { /* ... */ },
  onError: function(error, payment) { /* ... */ },
});

```

This code block is a **simplified example** to give you a sense of how it works.

In order to make sure that all involved parties (your app, your server, the Pi servers, and the Pi blockchain) are in sync,
the payment needs to go through a **Server-Side Approval** flow and a **Server-Side Completion** flow.

Please refer to:
* [the full Payments documentation](./payments.md) to learn about the complete payment flow
* [the Platform API documentation](./platform_API.md) to learn how to confirm the payment and acknowledge it from your
  server
* [the client SDK documentation](./SDK_reference.md) to learn about Pi Apps SDK and provided methods in detail
* the Demo App (coming soon!) to view an example of how you can implement the various required flows in your app's code.
