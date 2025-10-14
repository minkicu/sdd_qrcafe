# 3. API Interface (OpenAPI 3.1)

```yaml
openapi: 3.1.0
info:
  title: Pop-Up Café QR Order & Pay API
  version: "1.0.0"
  description: |
    API for managing menus, orders, and payments for a pop-up café.
    - **Traceability ID**: `INT-API-V1`
    - **Error Policy**: Conforms to RFC 7807 (`application/problem+json`).
  contact:
    name: API Support
    url: https://example.com/support
servers:
  - url: https://api.popup.cafe/v1
    description: Production Server
  - url: http://localhost:3000/v1
    description: Local Development

tags:
  - name: Menu
    description: Operations for retrieving menu information.
  - name: Orders
    description: Operations for creating and managing customer orders.
  - name: Kitchen
    description: Operations for the kitchen display and staff.

# ===============================================================================
# Paths
# ===============================================================================

paths:
  /booths/{boothId}/menu:
    get:
      tags:
        - Menu
      summary: Get Booth Menu
      description: Retrieves the full menu with all available items and options for a specific booth.
      operationId: getMenuByBooth
      parameters:
        - name: boothId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        "200":
          description: Successfully retrieved the menu.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Menu"
        "404":
          $ref: "#/components/responses/NotFound"

  /orders:
    post:
      tags:
        - Orders
      summary: Create a New Order
      description: Allows a customer to place a new order. The system validates stock and calculates the final price.
      operationId: createOrder
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/CreateOrderRequest"
      responses:
        "201":
          description: Order created successfully. Returns the order's ID, queue number, and tracking URL.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/CreateOrderResponse"
        "400":
          $ref: "#/components/responses/BadRequest"
        "409":
          description: Conflict, e.g., an item is out of stock.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ErrorResponse"
              example:
                type: "https://errors.example.com/order-conflict"
                title: "Item Not Available"
                status: 409
                code: "ERR-ORD-002"
                detail: "Item 'Iced Latte' is currently out of stock."
                instance: "/orders"
        "422":
          $ref: "#/components/responses/UnprocessableEntity"

  /orders/tracking/{trackingToken}:
    get:
      tags:
        - Orders
      summary: Get Order Status by Token
      description: Retrieves the real-time status of an order using a secure tracking token. This is a public endpoint for customers.
      operationId: getOrderStatusByToken
      parameters:
        - name: trackingToken
          in: path
          required: true
          schema:
            type: string
      responses:
        "200":
          description: Successfully retrieved the order status.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/OrderStatusResponse"
        "404":
          $ref: "#/components/responses/NotFound"

  /kitchen/booths/{boothId}/orders:
    get:
      tags:
        - Kitchen
      summary: Get Active Orders for Kitchen
      description: Retrieves a list of active orders for the kitchen screen, with optional status filtering. Requires staff authentication.
      operationId: getKitchenOrders
      parameters:
        - name: boothId
          in: path
          required: true
          schema:
            type: string
            format: uuid
        - name: status
          in: query
          required: false
          schema:
            type: string
            enum: [Queued, In-Progress, Ready]
      responses:
        "200":
          description: A list of active orders.
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/KitchenOrder"
        "401":
          $ref: "#/components/responses/Unauthorized"
        "403":
          $ref: "#/components/responses/Forbidden"

  /kitchen/orders/{orderId}/status:
    put:
      tags:
        - Kitchen
      summary: Update Order Status
      description: Allows kitchen staff to update the status of an order.
      operationId: updateOrderStatus
      parameters:
        - name: orderId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                status:
                  type: string
                  enum: [In-Progress, Ready, Picked-up]
              required: ["status"]
      responses:
        "200":
          description: Status updated successfully.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/KitchenOrder"
        "400":
          $ref: "#/components/responses/BadRequest"
        "401":
          $ref: "#/components/responses/Unauthorized"
        "403":
          $ref: "#/components/responses/Forbidden"
        "409":
          description: Invalid status transition or action.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ErrorResponse"
              example:
                type: "https://errors.example.com/state-transition-invalid"
                title: "Invalid Status Change"
                status: 409
                code: "ERR-ORD-003"
                detail: "Cannot change status from 'Picked-up' to 'Ready'."
                instance: "/kitchen/orders/{orderId}/status"

# ===============================================================================
# Components
# ===============================================================================

components:
  securitySchemes:
    KitchenAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT token for kitchen staff and booth owners.

  schemas:
    # --- Main Models ---
    Menu:
      type: object
      properties:
        boothId:
          type: string
          format: uuid
        items:
          type: array
          items:
            $ref: "#/components/schemas/MenuItem"
    MenuItem:
      type: object
      properties:
        itemId:
          type: string
          format: uuid
        name:
          type: string
        description:
          type: string
        imageUrl:
          type: string
          format: uri
        basePrice:
          type: number
          format: float
        category:
          type: string
        isAvailable:
          type: boolean
        options:
          type: array
          items:
            $ref: "#/components/schemas/MenuOptionGroup"
    MenuOptionGroup:
      type: object
      properties:
        groupName:
          type: string
          example: "Size"
        options:
          type: array
          items:
            $ref: "#/components/schemas/MenuOption"
    MenuOption:
      type: object
      properties:
        optionName:
          type: string
          example: "Large"
        priceDelta:
          type: number
          format: float
          example: 1.50

    # --- Order Creation ---
    CreateOrderRequest:
      type: object
      required: [boothId, items, payment]
      properties:
        boothId:
          type: string
          format: uuid
        customerNickname:
          type: string
          maxLength: 50
        items:
          type: array
          minItems: 1
          items:
            $ref: "#/components/schemas/OrderItemRequest"
        payment:
          $ref: "#/components/schemas/PaymentRequest"
    OrderItemRequest:
      type: object
      required: [itemId, quantity]
      properties:
        itemId:
          type: string
          format: uuid
        quantity:
          type: integer
          minimum: 1
        options:
          type: array
          items:
            $ref: "#/components/schemas/SelectedOption"
    SelectedOption:
      type: object
      required: [groupName, optionName]
      properties:
        groupName:
          type: string
        optionName:
          type: string
    PaymentRequest:
      type: object
      required: [method]
      properties:
        method:
          type: string
          enum: [Mock, SlipUpload]
        slipUploadId:
          type: string
          description: "An identifier for a pre-uploaded slip file. Required if method is SlipUpload."

    CreateOrderResponse:
      type: object
      properties:
        orderId:
          type: string
          format: uuid
        queueNumber:
          type: integer
        trackingUrl:
          type: string
          format: uri

    # --- Order Status ---
    OrderStatusResponse:
      type: object
      properties:
        orderId:
          type: string
          format: uuid
        queueNumber:
          type: integer
        status:
          type: string
          enum: [Queued, In-Progress, Ready, Picked-up, Cancelled]
        customerNickname:
          type: string
        pickupCode:
          type: string
          description: "Shown to customer only when status is 'Ready'."
          example: "1234"
        items:
          type: array
          items:
            $ref: "#/components/schemas/OrderItemSnapshot"
    OrderItemSnapshot:
      type: object
      properties:
        name:
          type: string
        quantity:
          type: integer
        options:
          type: string
          example: "Size: Large, Milk: Oat"

    # --- Kitchen View ---
    KitchenOrder:
      allOf:
        - $ref: "#/components/schemas/OrderStatusResponse"
        - type: object
          properties:
            createdAt:
              type: string
              format: date-time

    # --- Generic Error ---
    ErrorResponse:
      type: object
      required: [type, title, status, code, detail]
      properties:
        type:
          type: string
          format: uri
          description: A URI reference that identifies the problem type.
        title:
          type: string
          description: A short, human-readable summary of the problem type.
        status:
          type: integer
          description: The HTTP status code.
        code:
          type: string
          description: An application-specific error code.
        detail:
          type: string
          description: A human-readable explanation specific to this occurrence of the problem.
        instance:
          type: string
          format: uri
          description: A URI reference that identifies the specific occurrence of the problem.

  # ===============================================================================
  # Reusable Responses
  # ===============================================================================

  responses:
    BadRequest:
      description: "Bad Request. The server cannot or will not process the request due to something that is perceived to be a client error (e.g., malformed request syntax)."
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            type: "https://errors.example.com/validation-error"
            title: "Invalid Input"
            status: 400
            code: "ERR-VAL-001"
            detail: "The 'quantity' field must be a positive integer."
            instance: "/orders"
    Unauthorized:
      description: "Unauthorized. The client must authenticate itself to get the requested response."
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            type: "https://errors.example.com/auth-required"
            title: "Authentication Required"
            status: 401
            code: "ERR-AUTH-001"
            detail: "A valid authentication token was not provided."
            instance: "/kitchen/booths/{boothId}/orders"
    Forbidden:
      description: "Forbidden. The client does not have access rights to the content."
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            type: "https://errors.example.com/forbidden-access"
            title: "Access Denied"
            status: 403
            code: "ERR-AUTH-002"
            detail: "You do not have permission to access orders for this booth."
            instance: "/kitchen/booths/{boothId}/orders"
    NotFound:
      description: "Not Found. The server can not find the requested resource."
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            type: "https://errors.example.com/not-found"
            title: "Resource Not Found"
            status: 404
            code: "ERR-GEN-404"
            detail: "The requested booth or order could not be found."
            instance: "/booths/unknown-id/menu"
    UnprocessableEntity:
      description: "Unprocessable Entity. The server understands the content type of the request entity, but was unable to process the contained instructions (e.g., semantic errors)."
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            type: "https://errors.example.com/semantic-error"
            title: "Unprocessable Request"
            status: 422
            code: "ERR-ORD-001"
            detail: "Payment method 'SlipUpload' requires a 'slipUploadId'."
            instance: "/orders"
```
