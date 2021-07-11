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
    muchumacbd039: string
  }
}
```

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
