# 📑 Pop-up Café QR Order & Pay — MVP Requirements

## 0) Executive Summary
Customer scans a QR code at the table/booth → selects menu items → pays → kitchen receives the order in real time → customer tracks status and gets notified when the order is ready.  
MVP Goal: Fast order flow, reduced booth queue, no real payment integration (mock or slip upload).

---

## 1) Goals & Success Metrics
- **Goals**
  - Reduce average time from scan to order completion **< 90s**
  - Reduce booth queue **≥ 50%** vs. paper orders
  - Kitchen sees new order within **≤ 2s** of confirmation
- **Success Metrics**
  - Conversion scan → successful order ≥ **60%**
  - Order errors (wrong/missing items) **< 2%**
  - Downtime during booth operation **< 1%**

---

## 2) Scope (MVP)
### In-Scope
- Scan QR → open booth/table menu (no login)
- Select menu + size + toppings + quantity (with instant pricing)
- Cart (edit quantity/remove)
- Payment: **Mock Payment** or **Upload Transfer Slip**
- Order status page (Queued → In-Progress → Ready → Picked-up)
- Kitchen Screen: real-time order queue + change status
- Order confirmation/ticket number (on screen + pickup QR)

---

## 3) Personas & Primary Use Cases
- **Customer (Guest)**: Scan → browse menu → pay → track order → pickup  
- **Barista/Kitchen Staff**: View orders → prepare drinks → update status → call customer  
- **Booth Owner**: Open/close menu, set prices, view daily orders  

---

## 4) User Stories (MoSCoW)
**Must Have**
- As a customer, I want to scan a QR code and immediately see the booth menu.  
- I can select menu items + options (size, milk, toppings) and see updated price.  
- I can edit/remove items in the cart before confirming.  
- I can pay via mock or upload slip.  
- I receive a queue number + tracking link.  
- I can track real-time status until “Ready”.  
- Kitchen sees new orders instantly and updates status.  

**Should Have**
- Menu categories + search  
- Enter nickname for pickup  
- Prevent ordering out-of-stock items  

**Could Have**
- Booth theme colors/logo  
- “Reorder last” button in same session  

**Won’t Have (MVP)**
- Membership / coupons  
- Advanced analytics  

---

## 5) Functional Requirements
### 5.1 QR → Menu Routing
- Customers can scan a QR code at a booth or table to directly access the digital menu.

### 5.2 Menu & Options
- Menu = `name`, `image`, `price base`, `options`, `availability`  
- Options may include **price delta** (+10 for large size).

### 5.3 Cart & Pricing
- Customers can select products and add them to the shopping cart with chosen options (size, quantity, toppings).
- Add/remove/edit cart items  
- Show **total** (base + options + tax if any)  
- Stock validation  

### 5.4 Checkout & Payment
- The system supports payments via multiple channels (e.g., credit card, mobile banking, e-wallet, mock for MVP).
- On confirm:  
  - Create **Order** (status = `Queued`)  
  - Assign **Queue Number** (per booth)  
- Once payment is successful, customers receive a digital receipt and can track order status in real time.

### 5.5 Order Tracking
- Unique URL for tracking (guest access)  
- Status: `Queued` → `In-Progress` → `Ready` → `Picked-up`  
- At `Ready`: show pickup QR or 4-digit code  

### 5.6 Kitchen Screen
- Show orders sorted by time  
- Update status buttons  
- Filters: All / Queued / In-Progress / Ready  
- Show options (size/topping) clearly  

### 5.7 Pickup Verification
- Barista scans pickup QR or enters code → set `Picked-up`  
- Prevent duplicate pickup (idempotency)

### 5.8 Feedback System
- Customers can provide feedback after order completion to help booth owners improve service and product quality.


## 6) Non-Functional Requirements
- **Performance**: Kitchen receives new order **≤ 2s**  
- **Availability**: ≥ 99% uptime during booth open hours  
- **Usability**: Menu page loads in **≤ 2s** on 4G  
- **Compatibility**: Mobile-friendly (iOS/Android, Safari/Chrome latest)  
- **Security**: Tokenized order URLs, file upload restrictions  
- **Observability**: Log orders, status changes, errors  

---

## 7) Constraints & Assumptions
- Use **Mock Payment/Slip Upload** in MVP  
- Booth has stable internet (4G/Hotspot)  
- Guest checkout only (no accounts)  
- QR valid only for the event/booth session  

---
