@@ -56,24 +56,25 @@
  // An explanation of the payment - will be shown to the user:		  // An explanation of the payment - will be shown to the user:
  memo: "...", // e.g: "Digital kitten #1234",		  memo: "...", // e.g: "Digital kitten #1234",
  // An arbitrary developer-provided metadata object - for your own usage:		  // An arbitrary developer-provided metadata object - for your own usage:
  metadata: { /* ... */ }, // e.g: { kittenId: 1234 }		  metadata: { /* ... */ }, // e.g: { kittenId: 1234 }
}, {		}, {
  // Callbacks you need to implement - read more about those in the detailed docs linked below:		  // Callbacks you need to implement - read more about those in the detailed docs linked below:
  onReadyForServerApproval: function(paymentId) { /* ... */ },		  onReadyForServerApproval: function(paymentId) { /* ... */ },
  onReadyForServerCompletion: function(paymentId, txid) { /* ... */ },		  onReadyForServerCompletion: function(paymentId, txid) { /* ... */ },
  onCancel: function(paymentId) { /* ... */ },		  onCancel: function(paymentId) { /* ... */ },
  onError: function(error, payment) { /* ... */ },		  onError: function(error, payment) { /* ... */ },
});		});
```		```
This code block is a **simplified example** to give you a sense of how it works.		This code block is a **simplified example** to give you a sense of how it works.
In order to make sure that all involved parties (your app, your server, the Pi servers, and the Pi blockchain) are in sync,		In order to make sure that all involved parties (your app, your server, the Pi servers, and the Pi blockchain) are in sync,
the payment needs to go through a **Server-Side Approval** flow and a **Server-Side Completion** flow.		the payment needs to go through a **Server-Side Approval** flow and a **Server-Side Completion** flow.
Please refer to:		Please refer to:
 * [the full Payments documentation](./payments.md) to learn about the complete payment flow		 * [the full Payments documentation](./payments.md) to learn about the complete payment flow
 * [the Platform API documentation](./platform_API.md) to learn how to confirm the payment and acknowledge it from your		 * [the Platform API documentation](./platform_API.md) to learn how to confirm the payment and acknowledge it from your
   server		   server
 * [the client SDK documentation](./SDK_reference.md) to learn about Pi Apps SDK and provided methods in detail
 * the Demo App (coming soon!) to view an example of how you can implement the various required flows in your app's code.		 * the Demo App (coming soon!) to view an example of how you can implement the various required flows in your app's code.
