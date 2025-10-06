# 2. System Design

This document outlines the architecture, data models, and design principles for the Pop-up Café QR Order & Pay system.

## 1. Architecture Overview
The system will follow a simple, scalable client-server architecture.

- **Frontend (Client)**: A mobile-first Single Page Application (SPA) built with a modern web framework (e.g., React, Vue.js). It will be responsible for rendering the menu, cart, and order status pages. It communicates with the backend via a RESTful API.
- **Backend (Server)**: A monolithic API service that handles all business logic, including menu management, order processing, and status updates. It exposes REST endpoints for the frontend and the kitchen display.
- **Database**: A relational database (e.g., PostgreSQL) to store data related to menus, items, orders, and payments.
- **Real-time Layer**: A WebSocket connection will be used to push real-time order updates to the Kitchen Screen and potentially to the customer's tracking page.

### Data Flow
1.  **Menu Request**: Customer scans QR -> Frontend requests menu from Backend -> Backend fetches from DB.
2.  **Order Placement**: Customer submits order -> Frontend sends order payload to Backend -> Backend validates, stores in DB, and returns an order tracking URL.
3.  **Kitchen Update**: Backend publishes a "new order" event via WebSocket -> Kitchen Screen frontend receives the event and updates its display.
4.  **Status Update**: Kitchen staff changes status -> Kitchen Screen sends update request to Backend -> Backend updates DB and publishes "status change" event.
5.  **Customer Tracking**: Customer's tracking page frontend receives the "status change" event and updates the UI.

*(Traceability: REQ-001, REQ-006, REQ-007, REQ-011, REQ-012)*

## 2. Data Model

### Menu / Item
| Field | Type | Constraints | Description |
|---|---|---|---|
| `itemId` | UUID | Primary Key | Unique identifier for a menu item. |
| `boothId` | UUID | Foreign Key | The booth this item belongs to. |
| `name` | String | Not Null | "Espresso", "Latte" |
| `description`| Text | Nullable | "Rich dark roast with notes of chocolate." |
| `imageUrl` | String | Nullable | URL for the item's image. |
| `basePrice` | Decimal | Not Null | The starting price. |
| `category` | String | Nullable | "Coffee", "Pastries" |
| `isAvailable`| Boolean | Not Null | `true` if in stock, `false` otherwise. |

### ItemOption
| Field | Type | Constraints | Description |
|---|---|---|---|
| `optionId` | UUID | Primary Key | Unique ID for the option. |
| `itemId` | UUID | Foreign Key | The item this option belongs to. |
| `groupName` | String | Not Null | "Size", "Milk" |
| `optionName` | String | Not Null | "Large", "Oat Milk" |
| `priceDelta` | Decimal | Not Null | Price difference (`+1.00`, `-0.50`, `0.00`). |

### Order
| Field | Type | Constraints | Description |
|---|---|---|---|
| `orderId` | UUID | Primary Key | Unique identifier for the order. |
| `boothId` | UUID | Foreign Key | The booth that received the order. |
| `queueNumber` | Integer | Not Null | Daily sequential number (e.g., 1, 2, 3). See GAP-02. |
| `trackingToken`| String | Unique, Not Null | Secure token for the guest tracking URL. |
| `status` | Enum | Not Null | `Queued`, `In-Progress`, `Ready`, `Picked-up`, `Cancelled`. |
| `totalPrice` | Decimal | Not Null | The final calculated price of the order. |
| `customerNickname`|String| Nullable | Nickname for pickup. |
| `pickupCode` | String | Not Null | 4-digit code for verification. |
| `createdAt` | Timestamp | Not Null | |
| `updatedAt` | Timestamp | Not Null | |

### OrderItem
| Field | Type | Constraints | Description |
|---|---|---|---|
| `orderItemId`| UUID | Primary Key | Unique ID for this line item. |
| `orderId` | UUID | Foreign Key | The order this item belongs to. |
| `itemId` | UUID | Foreign Key | The menu item that was ordered. |
| `quantity` | Integer | Not Null, > 0 | Number of this item ordered. |
| `unitPrice` | Decimal | Not Null | The price for a single unit, including options. |
| `options` | JSONB | Nullable | A snapshot of selected options, e.g., `[{"group": "Size", "option": "Large"}]`. |

*(Traceability: REQ-002, REQ-004, REQ-006, REQ-008)*

## 3. Interfaces
- **Client-Backend API**: A RESTful API over HTTPS. All data is exchanged in JSON format. This is the primary interface, defined in `03-interface.md`.
- **Backend-Kitchen Screen**: A WebSocket channel for real-time updates. The Kitchen Screen will subscribe to an `orders:{boothId}` channel. The backend will push new orders and status updates as JSON payloads.

*(Traceability: REQ-011, REQ-012)*

## 4. Security Considerations
- **Authentication**: No user authentication is required for customers (guest checkout). Booth Owners and Kitchen Staff will authenticate via a simple, secure mechanism (e.g., JWT tokens obtained via a separate login flow, out of scope for this spec).
- **Authorization**:
    - Customers can only view their own order via the secret tracking URL.
    - Kitchen Staff can only update orders for their assigned booth.
- **Data Protection**:
    - All communication is over HTTPS.
    - Payment slip uploads will be scanned for malware, and access will be restricted. File types will be limited to common image formats (`JPEG`, `PNG`). (REQ-106)
    - Order tracking URLs will use a high-entropy, unguessable token. (REQ-105)

## 5. Error Handling
- The system will use standard HTTP status codes and the `application/problem+json` format for all error responses, as mandated by the `readme.md`.
- **Client-side errors** (e.g., invalid input) will be met with `4xx` responses.
- **Server-side errors** (e.g., database connection failure) will trigger `5xx` responses.
- Specific error codes (`ERR-###`) will provide granular detail for logging and client-side error handling.

## 6. Sequence / Flow Diagram (Order Placement)

```mermaid
sequenceDiagram
    participant C as Customer (Frontend)
    participant S as Server (Backend)
    participant DB as Database
    participant K as Kitchen (Frontend)

    C->>S: POST /orders (items, options, paymentInfo)
    activate S
    S->>S: Validate stock for all items (REQ-016)
    alt Stock Unavailable
        S-->>C: 409 Conflict (ERR-ORD-002: Item out of stock)
    end
    S->>S: Calculate total price
    S->>DB: BEGIN TRANSACTION
    S->>DB: INSERT into Orders table (status='Queued')
    S->>DB: INSERT into OrderItems table
    S->>DB: COMMIT TRANSACTION
    S->>S: Generate trackingToken and pickupCode
    S-->>C: 201 Created (orderId, trackingUrl, queueNumber)
    deactivate S

    S-))K: WebSocket PUSH (new_order: {order details})
    activate K
    K->>K: Display new order on screen
    deactivate K
```

*(Traceability: REQ-005, REQ-006, REQ-011, REQ-016)*
