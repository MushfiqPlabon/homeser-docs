# Payment Flow API

This document describes the API flow for the payment processing system in the HomeSer service marketplace.

## Overview

The payment flow enables customers to securely process payments for bookings through SSLCommerz integration. The system handles payment initiation, status verification, refund processing, and transaction tracking with real-time updates for status changes.

## Flow Overview

1. Customer initiates payment for a booking
2. Payment session is created with SSLCommerz
3. Customer redirected to SSLCommerz payment gateway
4. Payment processed by SSLCommerz
5. Payment status verified through callback/webhook
6. Booking status updated based on payment result
7. Real-time updates notify of payment status changes

## API Endpoints

### Create Payment
- **Method**: POST
- **Endpoint**: `/api/payments/payments/`
- **Request**:
```json
{
  "booking_id": 123,
  "amount": 150.00,
  "currency": "BDT"
}
```
- **Response**:
```json
{
  "payment_id": 456,
  "transaction_id": "unique_sslcommerz_transaction_id",
  "redirect_url": "https://secure.sslcommerz.com/.../payment"
}
```
- **Authentication**: Required
- **Authorization**: Customer associated with booking
- **Business Logic**: Creates payment record with pending status and returns SSLCommerz redirect URL

### Payment Verification (Client-initiated)
- **Method**: POST
- **Endpoint**: `/api/payments/payments/{id}/verify/`
- **Request**:
```json
{
  "transaction_id": "transaction_id_from_sslcommerz"
}
```
- **Response**: Verification result
- **Authentication**: Required
- **Authorization**: Customer associated with payment

### Get Payment Details
- **Method**: GET
- **Endpoint**: `/api/payments/payments/{id}/`
- **Request**: Payment ID in URL path
- **Response**: Complete payment details
- **Authentication**: Required
- **Authorization**: Customer or provider associated with linked booking

### Initiate Refund
- **Method**: POST
- **Endpoint**: `/api/payments/payments/{id}/initiate_refund/`
- **Request**:
```json
{
  "amount": 150.00,
  "reason": "Service cancellation"
}
```
- **Response**: Refund processing details
- **Authentication**: Required
- **Authorization**: Admin or authorized staff

### Webhook Endpoint (Server-to-Server)
- **Method**: POST
- **Endpoint**: `/api/payments/payments/webhook/`
- **Request**: SSLCommerz IPN (Instant Payment Notification) callback
- **Response**: Webhook processing status
- **Authentication**: Public (server-to-server communication)
- **Security**: SSLCommerz signature validation and transaction ID verification
- **Webhook Validation Process**:
  - Extract `verify_sign` and `verify_key` from webhook data
  - Validate presence of verification parameters
  - Verify signature using MD5(`SSLCOMMERZ_STORE_PASS` + `verify_key`)
  - Compare computed signature with provided `verify_sign`
  - Only process webhook if signature validation passes
  - Implement replay protection using `webhook_processed_ids` JSONField

### Transaction History
- **Method**: GET
- **Endpoint**: `/api/payments/transactions/`
- **Request**: Query parameters for filtering
- **Response**: Paginated list of payment transactions
- **Authentication**: Required
- **Authorization**: Customer (own transactions), provider (own transactions), or admin

### User's Transactions
- **Method**: GET
- **Endpoint**: `/api/payments/transactions/my_transactions/`
- **Response**: Current user's transaction history
- **Authentication**: Required
- **Authorization**: Current user only

### Provider Earnings
- **Method**: GET
- **Endpoint**: `/api/payments/transactions/earnings/`
- **Response**: Provider's total earnings
- **Authentication**: Required
- **Authorization**: Provider role only

## SSLCommerz Integration

### Configuration
- Uses `sslcommerz_lib` Python library
- Configured with store ID and password from environment variables
- Supports both sandbox and production modes
- Payment validation through server-to-server communication

### Payment Session Creation
1. Create payment record in local database with pending status
2. Generate unique transaction ID
3. Create SSLCommerz session with booking details
4. Return payment URL for customer redirection

### Payment Verification Process
1. SSLCommerz sends callback to verification endpoint
2. System validates transaction through SSLCommerz API
3. Payment status updated in database
4. Booking status updated based on payment result
5. Real-time notifications sent to relevant users

## Error Handling

### 400 Bad Request
- **Code**: `PAYMENT_VALIDATION_ERROR`
- **Message**: Payment request validation failed

### 402 Payment Required
- **Code**: `PAYMENT_DECLINED`
- **Message**: Payment was declined by payment processor

### 409 Conflict
- **Code**: `PAYMENT_ALREADY_PROCESSED`
- **Message**: Payment has already been processed

### 502 Bad Gateway
- **Code**: `PAYMENT_GATEWAY_ERROR`
- **Message**: Error communicating with payment processor

## Security Considerations

- Server-to-server verification prevents client-side tampering
- Transaction ID validation to prevent duplicate processing
- Payment amount validation against original booking amount
- SSLCommerz signature verification for callback authenticity
- Rate limiting on payment endpoints to prevent abuse
- Secure storage of SSLCommerz credentials in environment variables
- Webhook replay protection using `webhook_processed_ids` JSONField to prevent duplicate processing
- Signature verification using MD5(store_password + verify_key) for authenticating webhook requests
- Validation of webhook transactions against existing database records before processing

## Refund Processing

### Refund Types
- **Full Refund**: Complete refund of original payment amount
- **Partial Refund**: Partial refund up to original payment amount
- **Service Fee Refund**: Refund of service fees if applicable

### Refund Validation
- Verify refund amount does not exceed original payment
- Check booking status before processing refund
- Validate refund reason and authorization
- Update related bookings and services appropriately

## Real-time Updates

### Payment Status Updates
- Channel: `payments:{payment_id}`
- Event: `payment.status_updated`
- Payload: Updated payment status and details

### Booking Status Updates
- Channel: `bookings:{booking_id}`
- Event: `booking.status_updated`
- Payload: Booking status updated based on payment result

### Customer Notifications
- Channel: `notifications:{customer_id}`
- Event: `payment.status_change`
- Payload: Payment status change notification

## Performance Considerations

- Cache payment configuration to reduce external API calls
- Asynchronous processing for payment verification callbacks
- Proper database indexing for payment queries
- Efficient serialization for payment history retrieval

## Related Documentation

- [See: api/common-patterns.md]
- [See: security/security-measures.md]
- [See: api/booking-flow.md]
- [See: data/models.md]
- [See: api/real-time-events.md]

## Implementation Notes

- Payment processing uses a comprehensive refund system with status tracking
- The system supports multiple payment methods through SSLCommerz
- Transaction tracking ensures complete financial audit trail
- Payment validation includes both client-side and server-side verification
- Webhook security includes SSLCommerz signature verification