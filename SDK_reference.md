# Client SDK reference:

## Initialization

Add the following `script` tags to all pages where you need to call the Pi Apps SDK:

```html
<script src="https://sdk.minepi.com/pi-sdk.js"></script>
<script>Pi.init({ version: "2.0" })</script>
```

The config object passed to the init function accepts the following keys:
* `version` (string, required) - this is required to ensure compatibility of your app with newer SDK versions that might bring
  breaking changes (in which case breaking changes will be implemented under a higher version number)
* `sandbox`: (boolean, optional) - this enables you to configure the SDK to run in the sandbox.

### Using the SDK in sandbox mode:

```html
<script src="https://sdk.minepi.com/pi-sdk.js"></script>
<script>Pi.init({ version: "2.0", sandbox: true })</script>
```

You may now run your app in the sandbox environment (https://sandbox.minepi.com), provided you've configured
a development URL in the developer portal. Visit the developer portal by opening develop.pi in the Pi Browser
to configure this and view your Sandbox URL.

> Typically, if you're using a framework or a boilerplate that supports it, you should be able to set up your
> sandbox flag to match your development environment. For example, most good Node boilerplates will set up the
> value of `process.env.NODE_ENV` to either `"development"` or `"production"`, and you could do something like:
> `Pi.init({ version: "2.0", sandbox: <%= process.env.NODE_ENV !== 'production' %> })`. This depends on your
> setup, but running the Pi SDK in sandbox mode will generally happen whenever your app is running in development.


## Authentication

```typescript
Pi.authenticate(scopes: Array<string>, onIncompletePaymentFound: Function<PaymentDTO>): Promise<AuthResult>
```

Return value:

```typescript
type AuthResult = {
  accessToken: string,
  user: {
    uid: string,
    username: string,
    roles: Array<string>
  }
}
```

### `scopes`

Available scopes: `username`, `payments`, `roles`.

Here is a breakdown of various keys available on the `AuthResult['user']` object, and the scopes required for those keys
to be present:
> Notice that the `payments` scope doesn't have a corresponding field in the return value. This is to avoid confusion on the user-end when showing the consent popup. 
> In order for users to be able to make payments, you need to make sure to use `payments` scope.

| Field         | Description    | Required Scope  |
| -------------: | ------------- | :-------------: |
| `uid`      | An app-local identifier for the user. This is specific to this user, and this app. It will change if the user revokes the permissions they granted to your app. | (none) |
| `username`   | The user's Pi username.      |   `username` |
| `roles`      | The user's Pi community roles. Available values are `core_team`, `mega_mod`, `moderator` and `email_verified`. This field is subject to change in the future. | `roles`|

### `onIncompletePaymentFound`

Signature: `(payment: PaymentDTO) => void`

Every time the user is authenticated, and when they try to start a new payment flow, the SDK checks that there is no
incomplete payment for this user. An incomplete payment, in this context, is a payment which has been submitted to
the Pi blockchain, but where `status.developer_completed` is still `false` (i.e. the developer has not called the
`/complete` endpoint on this payment).

If an incomplete payment is found, this callback will be invoked with the payment's `PaymentDTO`.

When this callback is invoked, it is your responsibility to complete the corresponding payment (you should most
likely send the payment DTO to your server, and process it according to your business logic). You'll need to do
so before you can request a new payment from the user.


## Payments

Create a new payment:

```typescript
type PaymentData = {
  amount: number,
  memo: string,
  metadata: Object,
};

type PaymentCallbacks = {
  onReadyForServerApproval: (paymentId: string) => void,
  onReadyForServerCompletion: (paymentId: string, txid: string) => void,
  onCancel: (paymentId: string) => void,
  onError: (error: Error, payment?: PaymentDTO) => void,
};

Pi.createPayment(paymentData: PaymentData, callbacks: PaymentCallbacks): void;
```

The `createPayment` method takes two argument: `paymentData` and `callbacks`.

It will immediately start the payment flow, which will open on top of your app, enabling the user to review
the payment and submit the blockchain transaction, or reject it.

> **Warning: concurrent payments:**
>
> When creating a new payment, if there is already an open payment with your app for the current user:
> * If the user has not yet made the blockchain transaction, the open payment will be cancelled.
> * If the user has already made the blockchain transaction, the new payment will be rejected
> (`onError` will be called) and the `onIncompletePaymentFound` callback that was passed to the `authenticate`
> method will be called with the existing payment (use this callback to resolve the situation, e.g by sending
> the previous payment to your server for server-side completion).



### `paymentData` keys:

#### `amount`

This is the amount that the user is requested to pay to your app.

Example: `3.1415`.

#### `memo`

A memo for this payment. This will be visible to the user in the payment confirmation page.
Use this to briefly explain what the user is paying for.

Example: `Digital kitten #1234`.

#### `metadata`

An arbitrary Javascript object that you can attach to this payment. This is for your own use.
You should use this object as a way to link this payment with your internal
business logic.

Example: `{ orderId: 1234, itemIds: [11, 42, 314] }`

### `callbacks` keys:

#### `onReadyForServerApproval`

Signature: `(paymentId: string) => void`

This is called when the payment identifier (paymentId) is obtained from Pi Servers.

Use this callback to send the paymentId to your backend for **Server-Side Approval**.
Read more about Server-Side Approval and the full payment flow in the dedicated
[Payments](payments.md) page.

#### `onReadyForServerCompletion`

Signature: `(paymentId: string, txid: string) => void`

This is called when the user has submitted the transaction to the Pi blockchain.

Use this callback to send the blockchain transaction identifier (txid), along with the paymentId
to your backend for **Server-Side Completion**.
Read more about Server-Side Completion and the full payment flow in the dedicated
[Payments](payments.md) page.

#### `onCancel`

Signature: `(paymentId: string) => void`

This is called when the payment is cancelled (by a user action, or programmatically).

The payment may be cancelled either because the user manually rejected it, or because
of some other blocking situation: the user doesn't have enough funds on their account
to make the payment, another payment has been created concurrently...

#### `onError`

Signature: `(error: Error, payment?: PaymentDTO) => void`

This is called when an error occurs and the payment cannot be made. If the payment has been
created, the second argument will be present and you may use it to investigate the error.
Otherwise, only the first argument will be provided.


### Type `PaymentDTO`

This type is used in the arguments that are passed to `onIncompletePaymentFound` and `onError`.

```typescript
type PaymentDTO = {
  // Payment data:
  identifier: string, // The payment identifier
  user_uid: string, // The user's app-specific ID
  amount: number, // The payment amount
  memo: string, // A string provided by the developer, shown to the user
  metadata: Object, // An object provided by the developer for their own usage
  to_address: string, // The recipient address of the blockchain transaction
  created_at: string, // The payment's creation timestamp
  
  // Status flags representing the current state of this payment
  status: {
    developer_approved: boolean, // Server-Side Approval
    transaction_verified: boolean, // Blockchain transaction verified
    developer_completed: boolean, // Server-Side Completion
    cancelled: boolean, // Cancelled by the developer or by Pi Network
    user_cancelled: boolean, // Cancelled by the user
  },
  
  // Blockchain transaction data:
  transaction: null | { // This is null if no transaction has been made yet
    txid: string, // The id of the blockchain transaction
    verified: boolean, // True if the transaction matches the payment, false otherwise
    _link: string, // A link to the operation on the Blockchain API
  },
}
```


## Share dialog

Open a native share dialog:

```typescript
Pi.openShareDialog(title: string, message: string): void;
```

Use this method to open a native Share dialog (provided by the phone's OS), enabling your users to share
content from your app with their friends.

* `title`: the title of the message being shared
* `message`: the message that will be sent when the user picks a target app in the Share flow
