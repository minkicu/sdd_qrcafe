# 1. Requirements (EARS)

This document translates the project needs into formal EARS-style requirements.

## GAPS & Assumptions
- **GAP-01**: The exact tax calculation logic is not specified. (Assumption: No tax is applied in the MVP).
- **GAP-02**: The format for the "Queue Number" (e.g., numeric, alphanumeric, daily reset) is not defined. (Assumption: A simple integer that resets daily, e.g., 1, 2, 3...).
- **GAP-03**: The specific fields required for the "Booth Owner" persona (e.g., for viewing daily orders) are not detailed.
- **GAP-04**: The mechanism for "opening/closing" a menu by the Booth Owner is not defined. (Assumption: This is a manual backend operation for the MVP).
- **GAP-05**: The character set and length limits for a customer's "nickname" are not specified.
- **GAP-06**: Specific restrictions on uploaded payment slips (file types, size limits) are not defined.

---

## Functional Requirements

### Customer Flow
- **REQ-001**: When a Customer scans a QR code at a booth, the system shall immediately display the menu for that specific booth.
- **REQ-002**: While viewing a menu, the system shall allow the Customer to select an item, choose its options (e.g., size, toppings), and specify a quantity to add to their cart.
- **REQ-003**: If a Customer selects an item with options that modify the price, the system shall instantly update and display the item's total price.
- **REQ-004**: While the cart contains items, the system shall allow the Customer to edit the quantity of an item or remove it entirely.
- **REQ-005**: When a Customer proceeds to checkout, the system shall display the final total price and provide options for mock payment or payment slip upload.
- **REQ-006**: After the Customer confirms their order, the system shall create the order with a "Queued" status, assign a unique queue number, and provide a unique link to the order tracking page.
- **REQ-007**: When a Customer accesses their unique order tracking link, the system shall display the real-time status of their order (`Queued`, `In-Progress`, `Ready`, `Picked-up`).
- **REQ-008**: If a Customer enters a nickname during checkout, the system shall associate it with the order for pickup.
- **REQ-009**: When the order status changes to "Ready", the system shall display a pickup verification code (e.g., 4-digit code or QR code) on the tracking page.
- **REQ-010**: After an order is `Picked-up`, the system shall allow the customer to provide feedback.

### Kitchen / Staff Flow
- **REQ-011**: The system shall display all new, confirmed orders on the Kitchen Screen in real-time (sorted chronologically).
- **REQ-012**: While viewing the order list, the Kitchen Staff shall be able to change an order's status (e.g., from `Queued` to `In-Progress`, `In-Progress` to `Ready`).
- **REQ-013**: The system shall provide filters on the Kitchen Screen to view orders by status (`All`, `Queued`, `In-Progress`, `Ready`).
- **REQ-014**: When Kitchen Staff marks an order as `Picked-up` (e.g., by scanning the customer's pickup QR), the system shall ensure the action is idempotent and cannot be repeated for the same order.

### Booth Owner / Menu Management
- **REQ-015**: The system shall allow a Booth Owner to view a list of daily orders.
- **REQ-016**: The system shall prevent a Customer from ordering an item that is marked as "out-of-stock".
- **REQ-017**: The menu displayed to customers shall support categories and a search function.

---

## Non-Functional Requirements

- **REQ-101 (Performance)**: The system shall ensure a new order is visible on the Kitchen Screen within 2 seconds of successful customer confirmation.
- **REQ-102 (Performance)**: The system shall load the menu page within 2 seconds on a standard 4G mobile connection.
- **REQ-103 (Availability)**: The system shall maintain at least 99% uptime during the booth's specified operating hours.
- **REQ-104 (Usability)**: The system's user interface shall be mobile-friendly and compatible with the latest versions of Safari and Chrome on both iOS and Android.
- **REQ-105 (Security)**: The system shall use unguessable, tokenized URLs for guest order tracking pages.
- **REQ-106 (Security)**: The system shall enforce restrictions on payment slip uploads (e.g., file type, size) to prevent malicious uploads. (See GAP-06)
- **REQ-107 (Observability)**: The system shall log all order creation events, status changes, and payment attempts for monitoring and auditing.
