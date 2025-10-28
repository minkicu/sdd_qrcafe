# 4. Behavior & Edge Cases

This document lists potential edge cases, mapping them to requirements, error codes, and expected HTTP responses.

| Category | Case Description | REQ-ID(s) | ERR-ID | HTTP Status |
|---|---|---|---|---|
| **Validation** | Request body is not valid JSON. | REQ-005 | ERR-VAL-000 | 400 Bad Request |
| | `CreateOrderRequest` is missing the required `items` array. | REQ-005 | ERR-VAL-001 | 400 Bad Request |
| | An item in the `items` array has a `quantity` of 0 or less. | REQ-002 | ERR-VAL-002 | 400 Bad Request |
| | An `itemId` provided in an order does not exist in the menu. | REQ-002 | ERR-VAL-003 | 400 Bad Request |
| | An option provided for an item does not exist (e.g., "Extra Large" size). | REQ-002 | ERR-VAL-004 | 400 Bad Request |
| | `customerNickname` exceeds the maximum length of 50 characters. | REQ-008 | ERR-VAL-005 | 400 Bad Request |
| | Payment `method` is `SlipUpload` but `slipUploadId` is missing. | REQ-005 | ERR-ORD-001 | 422 Unprocessable Entity |
| **Business** | Customer tries to order an item where `isAvailable` is `false`. | REQ-016 | ERR-ORD-002 | 409 Conflict |
| | Kitchen staff tries to change an order status in an invalid sequence (e.g., `Ready` to `Queued`). | REQ-012 | ERR-ORD-003 | 409 Conflict |
| | Kitchen staff tries to update the status of an already `Picked-up` or `Cancelled` order. | REQ-012, REQ-014 | ERR-ORD-004 | 409 Conflict |
| | A customer attempts to view an order using an invalid or expired `trackingToken`. | REQ-007 | ERR-GEN-404 | 404 Not Found |
| | A request is made to a `boothId` that does not exist. | REQ-001 | ERR-GEN-404 | 404 Not Found |
| **Security** | A request to a kitchen endpoint is made without a valid JWT. | REQ-011, REQ-012 | ERR-AUTH-001 | 401 Unauthorized |
| | A kitchen staff member from Booth A tries to access orders for Booth B. | REQ-011, REQ-012 | ERR-AUTH-002 | 403 Forbidden |
| **Idempotency** | A customer double-clicks the "Place Order" button, sending the same `CreateOrderRequest` twice. | REQ-005 | - | 201 (First), 200/409 (Second)* |
| | Kitchen staff attempts to scan the same customer pickup QR code twice. | REQ-014 | ERR-ORD-004 | 409 Conflict |

---
*\*Note on Order Idempotency: True idempotency for `POST /orders` would require a client-generated `Idempotency-Key`. For this MVP, if a duplicate request is detected (e.g., same payload within a short time), the server might return the original `201` response or a `409 Conflict` indicating a likely duplicate.*
