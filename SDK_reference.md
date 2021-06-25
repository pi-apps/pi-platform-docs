# Client SDK reference:

## Authentication

```typescript
Pi.authenticate(scopes: Array<string>, onIncompletePaymentFound: Function<PaymentDTO>): Promise<PiAuth>
```

Return value:

```typescript
type AuthResult {
  accessToken: string,
  user: {
    uid: string,
    pi_id: string,
    username: string
  }
}
```

### `scopes`

> **Not yet implemented**
>
> Currently, all calls to the `authenticate` method will assume all scopes have been requested.

Here is a breakdown of various keys available on the `PiAuth['user']` object, and the scopes required for those keys
to be present:

| Field         | Description    | Required Scope  |
| -------------: | ------------- | :-------------: |
| `uid`      | An app-local identifier for the user. This is specific to this user, and this app. It will change if the user revokes the permissions they granted to your app. | (none) |
| `username`   | The user's Pi username.      |   `username` |
| `pi_id`  | A network-wide identifier for this user. This will stay the same across all apps used by this user.      |   `username` |


### `onIncompletePaymentFound`

Signature: `(payment: PaymentDTO) => void`

Every time the user is authenticated, and when they try to start a new payment flow, the SDK checks that there is no incomplete payment for this user. An incomplete payment, in this context, is a payment which has been submitted to the Pi blockchain, but where `status.developer_completed` is still `false` (i.e. the developer has not called the `/complete` endpoint on this payment).

If an incomplete payment is found, this callback will be invoked with the payment's `PaymentDTO`.

When this callback is invoked, it is your responsibility to complete the corresponding payment (you should most likely send the payment DTO to your server, and process it according to your business logic). You'll need to do so before you can request a new payment from the user.


## Payments

Create a new payment:

```typescript
type PaymentData = {
  amount: number,
  reason: string,
  metadata: Object,
};

type PaymentCallbacks = {
  onPaymentIdReceived: (paymentId: string) => void,
  onTransactionSubmitted: (paymentId: string, txid: string) => void,
  onPaymentCancelled: (paymentId: string) => void,
  onPaymentError: (error: Error, payment?: PaymentDTO) => void,
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
> (`onPaymentError` will be called) and the `onOpenPaymentFound` method will be called with the
> existing payment (use this callback to resolve the situation, e.g by completing the previous
> payment from your backend).



## `paymentData` keys:

### `amount`

This is the amount that the user is requested to pay to your app.

Example: `3.1415`.

### `reason`

The reason for this payment. This will be visible to the user.

Example: `Please pay for order #1234`.

### `metadata`

An arbitrary object that you can attach to this payment. This is for your own use, and it won't
be shown to the user. You should use this object as a way to link this payment with your internal
business logic.

Example: `{ orderId: 1234, itemIds: [11, 42, 314] }`

## `callbacks` keys:

### `onPaymentIdReceived`

Signature: `(paymentId: string) => void`

This is called when the payment identifier (paymentId) is obtained from Pi Servers.

Use this callback to send the paymentId to your backend for **Server-Side Approval**.
Read more about Server-Side Approval and the full payment flow in the dedicated
[Payments](payments.md) page.

### `onTransactionSubmitted`

Signature: `(paymentId: string, txid: string) => void`

This is called when the user has submitted the transaction to the Pi blockchain.

Use this callback to send the blockchain transaction identifier (txid), along with the paymentId
to your backend for **Server-Side Completion**.
Read more about Server-Side Completion and the full payment flow in the dedicated
[Payments](payments.md) page.

### `onPaymentCancelled`

Signature: `(paymentId: string) => void`

This is called when the payment is cancelled (by a user action, or programmatically).

The payment may be cancelled either because the user manually rejected it, or because
of some other blocking situation: the user doesn't have enough funds on their account
to make the payment, another payment has been created concurrently...

### `onPaymentError`

Signature: `(error: Error, payment?: PaymentDTO) => void`

This is called when an error occurs and the payment cannot be made. If the payment has been
created, the second argument will be present and you may use it to investigate the error.
Otherwise, only the first argument will be provided.


## Share dialog

Open a native share dialog:

```typescript
Pi.openShareDialog(title: string, message: string): void;
```

Use this method to open a native Share dialog (provided by the phone's OS), enabling your users to share
content from your app with their friends.

* `title`: the title of the message being shared
* `message`: the message that will be sent when the user picks a target app in the Share flow
