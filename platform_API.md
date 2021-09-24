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

Retrieve the user's information.


```
GET /me
```

* Authorization method: **Access token**
* Response type: [UserDTO](#UserDTO)

Verify the data obtained with the frontend SDK (a malicious user could tamper with the requests and
send you wrong data) by sending the user’s access token to your backend and using this API endpoint
to verify the token validity and the user’s identity.

Access tokens are long, random strings, and the request will fail (401 HTTP error code) if the token
has been tampered with (a tampered token would
not belong to any real user).


### Payments

Base path: `/payments`.

#### Create a payment:

Do not create payments using the Platform API. Use the client-side Javascript SDK for this purpose.

#### Get a payment:

Get information about a payment.

```
GET /payments/{payment_id}
```

* Authorization method: **Server API Key**
* Response type: [PaymentDTO](#PaymentDTO)

#### Approve a payment:

Server-side approval: mark a payment as approved, enabling the user to submit the transaction to the blockchain.

```
POST /payments/{payment_id}/approve
```

* Authorization method: **Server API Key**
* Response type: [PaymentDTO](#PaymentDTO)

#### Complete a payment:

Server-side completion: mark a payment as completed by proving to the Pi Servers that your app has obtained the
payment's txid, enabling the user to close the payment flow.

```
POST /payments/{payment_id}/complete
```


* Authorization method: **Server API Key**
* Response type: [PaymentDTO](#PaymentDTO)

Example request body:

```
{
  "txid": "7a7ed20d3d72c365b9019baf8dc4c4e3cce4c08114d866e47ae157e3a796e9e7"
}
```

## Resource types

### `UserDTO`

```typescript
{
  "uid": string, // An app-specific user identifier
  "username": string, // The user's Pi username. Requires the `username` scope.
}
```

### `PaymentDTO`

```typescript
{
  // Payment data:
  "identifier": string, // The payment identifier
  "user_uid": string, // The user's app-specific ID
  "amount": number, // The payment amount
  "memo": string, // A string provided by the developer, shown to the user
  "metadata": Object, // An object provided by the developer for their own usage
  "to_address": string, // The recipient address of the blockchain transaction
  "created_at": string, // The payment's creation timestamp
  
  // Status flags representing the current state of this payment
  "status": {
    "developer_approved": boolean, // Server-Side Approval
    "transaction_verified": boolean, // Blockchain transaction verified
    "developer_completed": boolean, // Server-Side Completion
    "cancelled": boolean, // Cancelled by the developer or by Pi Network
    "user_cancelled": boolean, // Cancelled by the user
  },
  
  // Blockchain transaction data:
  "transaction": null | { // This is null if no transaction has been made yet
    "txid": string, // The id of the blockchain transaction
    "verified": boolean, // True if the transaction matches the payment, false otherwise
    "_link": string, // A link to the operation on the Blockchain API
  },
};
```
