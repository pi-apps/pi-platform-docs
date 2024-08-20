# Client SDK reference

## Initialization

Add the following `script` tags to all pages where you need to call the Pi Apps SDK:

```html
<script src="https://sdk.minepi.com/pi-sdk.js"></script>
<script>
  Pi.init({ version: "2.0" });
</script>
```

The config object passed to the init function accepts the following keys:

- `version` (string, required) - this is required to ensure compatibility of your app with newer SDK versions that might bring
  breaking changes (in which case breaking changes will be implemented under a higher version number)
- `sandbox`: (boolean, optional) - this enables you to configure the SDK to run in the sandbox.

### Using the SDK in sandbox mode:

```html
<script src="https://sdk.minepi.com/pi-sdk.js"></script>
<script>
  Pi.init({ version: "2.0", sandbox: true });
</script>
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

> **Warning:** The user information obtained with this method should not be passed to your backend and should
> only be used for presentation logic (e.g displaying the user’s username).
> **On your backend, use the Platform API as the source of truth.** You can verify the user's
> identity by requesting the /me endpoint from your backend, using the access token obtained with this method.

```typescript
Pi.authenticate(scopes: Array<Scope>, onIncompletePaymentFound: Function<PaymentDTO>): Promise<AuthResult>
```

Return value:

```typescript
type AuthResult = {
  accessToken: string;
  user: {
    uid: string;
    username: string;
  };
};
```

### `scopes`

Available scopes: `username`, `payments`, `wallet_address`

Here is a breakdown of various keys available on the `AuthResult['user']` object, and the scopes required for those keys
to be present:

|            Field | Description                                                                                                                                                     |  Required Scope  |
| ---------------: | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------: |
|            `uid` | An app-local identifier for the user. This is specific to this user, and this app. It will change if the user revokes the permissions they granted to your app. |      (none)      |
|       `username` | The user's Pi username.                                                                                                                                         |    `username`    |
|       `payments` | Required if your app needs to make payments between your app and the users                                                                                      |    `payments`    |
| `wallet_address` | The wallet address of the user who authenticates on your app.                                                                                                   | `wallet_address` |

### `onIncompletePaymentFound`

Signature: `(payment: PaymentDTO) => void`

Every time when the user is authenticated, and when they try to start a new payment flow, the SDK checks that there is no
incomplete payment for this user. An incomplete payment, in this context, is a payment which has been submitted to
the Pi Blockchain, but where `status.developer_completed` is still `false` (i.e. the developer has not called the
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
>
> - If the user has not yet made the blockchain transaction, the open payment will be cancelled.
> - If the user has already made the blockchain transaction, the new payment will be rejected
>   (`onError` will be called) and the `onIncompletePaymentFound` callback that was passed to the `authenticate`
>   method will be called with the existing payment (use this callback to resolve the situation, e.g by sending
>   the previous payment to your server for server-side completion).

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

This is called when the payment identifier (paymentId) is obtained from Pi Servers. During the approval time period, this callback will be invoked multiple times in case of failure.
If the initial trial fails, the Pi SDK will continue to invoke the function roughly every 10 seconds until the approval timer ends.

Use this callback to send the paymentId to your backend for **Server-Side Approval**.
Read more about Server-Side Approval and the full payment flow in the dedicated
[Payments](payments.md) page.

#### `onReadyForServerCompletion`

Signature: `(paymentId: string, txid: string) => void`

This is called when the user has submitted the transaction to the Pi Blockchain. During the completion time period, this callback will be invoked multiple times in case of failure.
If the initial trial fails, the Pi SDK will continue to invoke the function roughly every 10 seconds until the completion timer ends.

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

This type is for the arguments that are passed to `onIncompletePaymentFound` and `onError`.

```typescript
type PaymentDTO = {
  // Payment data:
  identifier: string; // payment identifier
  user_uid: string; // user's app-specific ID
  amount: number; // payment amount
  memo: string; // a string provided by the developer, shown to the user
  metadata: Object; // an object provided by the developer for their own usage
  from_address: string; // sender address of the blockchain transaction
  to_address: string; // recipient address of the blockchain transaction
  direction: Direction; // direction of the payment
  created_at: string; // payment's creation timestamp
  network: AppNetwork; // a network of the payment

  // Status flags representing the current state of this payment
  status: {
    developer_approved: boolean; // Server-Side Approval
    transaction_verified: boolean; // blockchain transaction verified
    developer_completed: boolean; // server-Side Completion
    cancelled: boolean; // cancelled by the developer or by Pi Network
    user_cancelled: boolean; // cancelled by the user
  };

  // Blockchain transaction data:
  transaction: null | {
    // This is null if no transaction has been made yet
    txid: string; // id of the blockchain transaction
    verified: boolean; // true if the transaction matches the payment, false otherwise
    _link: string; // a link to the operation on the Blockchain API
  };
};
```

### Type `Direction`

A developer can check the direction of the payment with this type.

```typescript
type Direction = "user_to_app" | "app_to_user";
```

### Type `AppNetwork`

Shows which network the payment is being made on.

```typescript
type AppNetwork = "Pi Network" | "Pi Testnet";
```

### Type `Scope`

Scopes you can request to users.

```typescript
type Scope = "username" | "payments" | "wallet_address";
```

## Native features list

Use this method to get a list of native features available for specific version of Pi Browser your user is using. Some SDK features may require particular features to work properly.

```typescript
type NativeFeature = "inline_media" | "request_permission" | "ad_network";

Pi.nativeFeaturesList(): Promise<Array<NativeFeature>>;
```

## Share dialog

Open a native share dialog:

```typescript
Pi.openShareDialog(title: string, message: string): void;
```

Use this method to open a native Share dialog (provided by the phone's OS), enabling your users to share
content from your app with their friends.

- `title`: the title of the message being shared
- `message`: the message that will be sent when the user picks a target app in the Share flow

## Ads

SDK contains the `Ads` module (object) which allows developers to display ads to the users. It provides several methods for both interstitial and rewarded ads. All of the methods take advantage of Promise-based asynchronicity.

```typescript
type AdType = "interstitial" | "rewarded";
```

### Show ad

Use `showAd(adType: "interstitial" | "rewarded")` method to display an ad to a user. It returns a promise which resolves with an object of type `ShowAdResponse`:

```typescript
type ShowAdResponse =
  | {
      type: "interstitial";
      result: "AD_CLOSED" | "AD_DISPLAY_ERROR" | "AD_NETWORK_ERROR" | "AD_NOT_AVAILABLE";
    }
  | {
      type: "rewarded";
      result: "AD_REWARDED" | "AD_CLOSED" | "AD_DISPLAY_ERROR" | "AD_NETWORK_ERROR" | "AD_NOT_AVAILABLE" | "ADS_NOT_SUPPORTED" | "USER_UNAUTHENTICATED";
      adId?: string;
    };

Pi.Ads.showAd(adType: AdType): Promise<ShowAdResponse>

```

- `"AD_NOT_AVAILABLE"` indicates the ad failed to load,
- `"AD_CLOSED"` indicates the ad was successfully displayed and closed,
- `"AD_REWARDED"` indicates the ad was successfully displayed and rewarded (only applicable for `rewarded` ads),
- `"AD_DISPLAY_ERROR"` indicates the ad was successfully loaded but failed to be displayed,
- `"AD_NETWORK_ERROR"` indicates that a user might have encountered network connection issues,
- `"ADS_NOT_SUPPORTED"` - indicates that app version used by uesr does not support ads,
- `"USER_UNAUTHENTICATED"` - indicates that user is not authenticated therefore rewarded ad cannot be display (only for `rewarded` ads).

The Pi Browser internally manages the ads availability strategy, automatically loading initial ads and reloading them whenever displayed. This ensures that ads are consistently ready for display. However, in rare cases, loading ads may be interrupted or the third party app may use `showAd()` methods too quickly (even before the next ad is ready). To allow developers handle these cases, the Pi SDK provides additional methods: `isAdReady()` and `requestAd()`.

</br>

> **Notes:**
>
> - If your application was approved to Pi Developer Ad Network, the response from `Pi.Ads.showAd('rewarded')` will contain additional `adId` fields, which allows you to [verify the rewarded status using Pi Platform API](ads.md#rewarded-ads-status-verification-with-pi-platform-api).
> - If your application wasn't approved to Pi Developer Ad Network, the `adId` field will be missing from `Pi.Ads.showAd('rewarded')` response. You should not grant rewards to users without verifying the `adId` using [Pi Platform API](platform_API.md#verify-a-rewarded-ad-status).

### Check if ad is ready

Use `isAdReady(adType: "interstitial" | "rewarded")` method to check if an ad of specific type is available. This method returns a promise which resolves with object of type `IsAdReadyResponse`.

```typescript
type IsAdReadyResponse = {
  type: "interstitial" | "rewarded";
  ready: boolean;
};

Pi.Ads.isAdReady(adType: AdType): Promise<IsAdReadyResponse>
```

The Pi Browser internally manages the ads availability strategy, automatically loading initial ads and reloading them whenever displayed. This ensures that ads are consistently ready for display. However, in rare cases, loading ads may be interrupted or the third party app may use `showAd()` methods too quickly (even before the next ad is ready). To address such cases, the Pi SDK provides this method to allow developers to check the availability of ads explicitly.

### Request ad

Use `requestAd(adType: "interstitial" | "rewarded")` method to request an interstitial or a rewarded ad. It returns a promise which resolves with an object of type `RequestAdResponse`:

```typescript
type RequestAdResponse = {
  type: "interstitial" | "rewarded";
  result: "AD_LOADED" | "AD_FAILED_TO_LOAD" | "AD_NOT_AVAILABLE";
};

Pi.Ads.requestAd(adType: AdType): Promise<RequestAdResponse>
```

As with the Ads availability strategy, the Pi Browser internally manages the process of requesting new ads to replace the displayed ones. While it's not guaranteed that an ad will be available at all time, developers can use `requestAd()` method to manually retry the ad request in case a promise returned by the `isAdReady()` resolved with `false`.
