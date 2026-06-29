# Platform API

The platform API allows you to read and write data to the Pi Servers related with your app deployed on the
Pi App Platform, and your app's users.

## Overview

### Base path:

The latest version of the Platform API is available at `api.minepi.com/v2`.

> **Note about API versioning:**
>
> The platform API is currently in v2.
> As much as possible, we will not make any breaking changes to a version of an API, and only release breaking changes as
> new major versions. However, we might make breaking changes to an existing version without notice, if those are
> necessary (e.g security or privacy fixes).

### Authorization

The Platform API supports two different authorization mechanisms.

#### Access token authorization

Some API calls require that you provide a user's access token to access the resource. They are generally related with
a user's data (e.g: `/me`). Those endpoints can be accessed using the following Authorization header:

```
Authorization: Bearer <user access token>
```

Those endpoints can be indifferently accessed from your backend / server app, or from your frontend / client app.

#### Server API Key authorization

For various reasons, some API calls must be made from your backend / server app.
Those endpoints can be accessed using the following Authorization header:

```
Authorization: Key <your Server API Key>
```

> **Warning: Server API keys are for servers only**
>
> Your Server API Key **must** be kept on your server, and must not be sent to clients (do not use it in your
> client javascript code).
> In the future, your server API key might enable sensitive operations on your app itself that your users should
> not be allowed to perform. Letting users access your server API key is a **serious security breach**.

## API Reference

### Authentication

#### Access a user's resource:

Retrieve the user's information, including user information limited to what the user has consented to share
with your app.

```
GET /me
```

- Authorization method: **Access token**
- Response type: [UserDTO](#UserDTO)

Verify the data obtained with the frontend SDK (a malicious user could tamper with the requests and
send you wrong data) by sending the user’s access token to your backend and using this API endpoint
to verify the token validity and the user’s identity.

Access tokens are long, random strings, and the request will fail (401 HTTP error code) if the token
has been tampered with (a tampered token would
not belong to any real user).

### Payments

There are two different payment types.

1. U2A (User-To-App)
2. A2U (App-To-User)

#### Create a payment (U2A):

If a payment type is U2A, use `createPayment` method of the client-side Javascript SDK to create a payment. You can refer to [Payments](./SDK_reference.md#payments) section for more details.

#### Create a payment (A2U):

```
POST /payments
```

- Authorization method: **Server API Key**
- Response type: [PaymentDTO](#PaymentDTO)

Example request body:

```
{
  "payment": {
    "amount": 1,
    "memo": "From app to user test",
    "metadata": {"test": "test metadata"},
    "uid": "a1111111-aaaa-bbbb-2222-ccccccc3333d"
  }
}
```

#### Get a payment:

Get information about a payment.

```
GET /payments/{payment_id}
```

- Authorization method: **Server API Key**
- Response type: [PaymentDTO](#PaymentDTO)

#### Approve a payment:

Server-side approval: mark a payment as approved, enabling the user to submit the transaction to the blockchain.

```
POST /payments/{payment_id}/approve
```

- Authorization method: **Server API Key**
- Response type: [PaymentDTO](#PaymentDTO)

#### Complete a payment:

Server-side completion: mark a payment as completed by proving to the Pi Servers that your app has obtained the
payment's txid, enabling the user to close the payment flow.

```
POST /payments/{payment_id}/complete
```

- Authorization method: **Server API Key**
- Response type: [PaymentDTO](#PaymentDTO)

Example request body:

```
{
  "txid": "7a7ed20d3d72c365b9019baf8dc4c4e3cce4c08114d866e47ae157e3a796e9e7"
}
```

#### Cancel a payment:

Mark the payment as cancelled.

```
POST /payments/{payment_id}/cancel
```

- Authorization method: **Server API Key**
- Response type: [PaymentDTO](#PaymentDTO)

#### Get incomplete server payments:

Returns the list of server payments (i.e A2U payments) which are in EITHER one of the two states below:

- payment created, but no blockchain transaction was made yet ;
- blockchain transaction submitted, but the payment has not been completed by the developer.

```
GET /payments/incomplete_server_payments
```

- Authorization method: **Server API Key**
- Response type: { "incomplete_server_payments": Array<[PaymentDTO](#PaymentDTO)> }

### Ads

#### Verify a rewarded ad status

Verify status of a rewarded ad by `adId` returned by client Pi SDK method `displayAd('rewarded')`

```
GET /ads_network/status/:adId
```

- Authorization method: **Server API Key**
- Response type: [RewardedAdStatusDTO](#RewardedAdStatusDTO)


### In-app notifications

In-app notifications let your server send notifications to users of your app.
Users will receive these notifications in the **Notification Center** screen inside the **Pi App**

> **Important: server-side only**
>
> This endpoint requires your **Server API Key**, so it must be called from your backend (do not call it directly from the client).
>
> In order for a user to receive in-app notifications from your app, your app must request the `in_app_notifications` scope using the Client SDK authentication flow (see [Client SDK: Authentication](./SDK_reference.md#authentication)).

#### Send notifications to users

Send one or more notifications to app users.

```
POST /v2/in_app_notifications/notify
```

- Authorization method: **Server API Key**
- Response type: `Object`

##### Limits

- Maximum **256** notifications per request.
- A given user may receive up to **48** notifications per **24 hours** from your app.

##### Request body

- `notifications` (array, required): list of notifications to send.
  - `title` (string, required): notification title.
  - `body` (string, required): notification body text.
  - `user_uid` (string, required): the recipient user id for your app.
  - `subroute` (string, required): a route within your app that the notification should open. Must start with `/`.

##### Example request body

```json
{
  "notifications": [
    {
      "title": "Test Notification",
      "body": "This is a test notification",
      "user_uid": "my-app-user-1111111-22222222-33333333",
      "subroute": "/shop"
    }
  ]
}
```

##### Example successful response

Note: Delivery is best-effort per user. Some notifications in the batch may not be delivered if the recipient has not granted consent for in-app notifications or has disabled them in the Pi App. In those cases, the corresponding items will not appear in `delivered_notifications`.

```json
{
  "success": true,
  "delivered_notifications": [
    {
      "id": "string",
      "title": "Test Notification",
      "body": "This is a test notification",
      "user_uid": "my-app-user-111111-222222-333333",
      "subroute": "/shop",
      "seen_at": null,
      "dismissed_at": null
    }
  ]
}
```

##### Errors

Error responses use the following format:

```json
{ "error": "string" }
```

##### Statuses

- `201 Created`: notifications delivered (see example response above).
- `400 Bad Request`: too many notifications in a single request (more than 256). Example message: `"Maximum 256 notifications allowed per request"`.
- `403 Forbidden`: notification quota exceeded for one or more users. Example message: `"Notification quota exceeded for users: <uid1>, <uid2>"`.
- `422 Unprocessable Entity`: missing required parameters or invalid notification payload.
- `500 Internal Server Error`: unexpected server error.


## Resource types

### `UserDTO`

```typescript
{
  uid: string, // An app-specific user identifier
  credentials: {
    scopes: Array<Scope>, // a list of granted scopes
    valid_until: {
      timestamp: number,
      iso8601: string
    }
  },
  username?: string, // The user's Pi username. Requires the `username` scope.
}
```

### `PaymentDTO`

```typescript
{
  // Payment data:
  identifier: string, // payment identifier
  user_uid: string, // user's app-specific ID
  amount: number, // payment amount
  memo: string, // a string provided by the developer, shown to the user
  metadata: Object, // an object provided by the developer for their own usage
  from_address: string, // sender address of the blockchain transaction
  to_address: string, // recipient address of the blockchain transaction
  direction: Direction, // direction of the payment
  created_at: string, // the payment's creation timestamp
  network: AppNetwork, // a network of the payment

  // Status flags representing the current state of this payment
  status: {
    developer_approved: boolean, // Server-Side Approval
    transaction_verified: boolean, // blockchain transaction verified
    developer_completed: boolean, // Server-Side Completion
    cancelled: boolean, // cancelled by the developer or by Pi Network
    user_cancelled: boolean, // cancelled by the user
  },

  // Blockchain transaction data:
  transaction: null | { // This is null if no transaction has been made yet
    txid: string, // id of the blockchain transaction
    verified: boolean, // true if the transaction matches the payment, false otherwise
    _link: string, // a link to the operation on the Blockchain API
  },
};
```

### `RewardedAdStatusDTO`

```typescript
{
  "identifier": string; // the adId token returned from the Pi SDK displayAd("rewarded") method
  "mediator_ack_status": "granted" | "revoked" | "failed" | null;
  "mediator_granted_at": string | null; // ISO 8601 date string
  "mediator_revoked_at": string | null; // ISO 8601 date string
}
```
