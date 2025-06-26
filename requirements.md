# 🛠️ Airbnb Clone – Backend Feature Requirements

This document outlines the technical and functional specifications for key backend features of the Airbnb Clone project. Each feature includes API endpoints, expected inputs and outputs, validation rules, and performance criteria.

---

## 🔐 1. User Authentication

### 🎯 Objective

Allow users (Guests and Hosts) to securely register, log in, and manage their authentication sessions.

### 🔌 API Endpoints

#### `POST /api/auth/register`

* **Purpose**: Register a new user
* **Input**:

  ```json
  {
    "name": "Jane Doe",
    "email": "jane@example.com",
    "password": "securePassword123",
    "role": "guest"
  }
  ```
* **Output**:

  ```json
  {
    "message": "User registered successfully",
    "token": "JWT_TOKEN_HERE"
  }
  ```
* **Validation Rules**:

  * Email must be valid and unique.
  * Password must be at least 8 characters.
  * Role must be either `"guest"` or `"host"`.

#### `POST /api/auth/login`

* **Input**:

  ```json
  {
    "email": "jane@example.com",
    "password": "securePassword123"
  }
  ```
* **Output**:

  ```json
  {
    "message": "Login successful",
    "token": "JWT_TOKEN_HERE"
  }
  ```

#### `GET /api/auth/profile`

* **Auth**: JWT token required
* **Output**:

  ```json
  {
    "name": "Jane Doe",
    "email": "jane@example.com",
    "role": "guest",
    "profileImage": "url_to_image"
  }
  ```

### ⚙️ Performance Criteria

* Passwords hashed with bcrypt.
* JWT expiry: 1 hour (with refresh support).
* Response time ≤ **200ms**.

---

## 🏠 2. Property Management

### 🎯 Objective

Enable hosts to create, update, delete, and manage property listings.

### 🔌 API Endpoints

#### `POST /api/properties`

* **Auth**: Host only
* **Input**:

  ```json
  {
    "title": "Cozy Beach House",
    "description": "Ocean view, 2 beds, pet-friendly.",
    "location": "Miami, FL",
    "price": 150,
    "amenities": ["Wi-Fi", "Pool", "Air Conditioning"],
    "images": ["img1.jpg", "img2.jpg"]
  }
  ```
* **Output**:

  ```json
  {
    "message": "Property created successfully",
    "propertyId": "123456"
  }
  ```
* **Validation Rules**:

  * Required: `title`, `location`, `price`, and at least one `image`.
  * Price must be a positive number.

#### `PUT /api/properties/:id`

* **Purpose**: Update a property
* **Auth**: Host (owner only)

#### `DELETE /api/properties/:id`

* **Purpose**: Delete a property listing
* **Auth**: Host (owner only)

### ⚙️ Performance Criteria

* Indexes on `location`, `price` for optimized filtering.
* Handle up to **100k+** listings with pagination and search optimizations.

---

## 📅 3. Booking System

### 🎯 Objective

Allow guests to book available properties and manage their reservations efficiently.

### 🔌 API Endpoints

#### `POST /api/bookings`

* **Auth**: Guest
* **Input**:

  ```json
  {
    "propertyId": "123456",
    "startDate": "2025-07-01",
    "endDate": "2025-07-05"
  }
  ```
* **Output**:

  ```json
  {
    "message": "Booking created",
    "bookingId": "b789"
  }
  ```
* **Validation Rules**:

  * Dates must not overlap with existing bookings.
  * End date must be after the start date.
  * Property must be available and active.

#### `GET /api/bookings/:id`

* **Purpose**: Retrieve a booking
* **Auth**: Guest or Host (who is part of the booking)

#### `DELETE /api/bookings/:id`

* **Purpose**: Cancel a booking
* **Auth**: Guest or Host (who is part of the booking)

### ⚙️ Performance Criteria

* Use database-level transactions to prevent double bookings.
* Optimized for real-time availability check.
* Booking creation response ≤ **300ms**.

---

## 💳 4. Payment Integration

### 🎯 Objective

Allow guests to securely pay for bookings and enable automatic payouts to hosts post-booking completion.

### 🔌 API Endpoints

#### `POST /api/payments/charge`

* **Auth**: Guest
* **Input**:

  ```json
  {
    "bookingId": "b789",
    "paymentMethod": "stripe",
    "currency": "USD"
  }
  ```
* **Output**:

  ```json
  {
    "message": "Payment successful",
    "transactionId": "txn_001"
  }
  ```
* **Validation Rules**:

  * Validate booking belongs to the user and is in `pending` status.
  * Validate booking dates are still available.
  * Payment method must be supported (e.g., `stripe`, `paypal`).

#### `GET /api/payments/history`

* **Auth**: Guest or Host
* **Purpose**: Show past payments and payouts
* **Output**:

  ```json
  [
    {
      "bookingId": "b789",
      "amount": 600,
      "status": "paid",
      "date": "2025-07-01"
    }
  ]
  ```

### ⚙️ Performance Criteria

* Use secure payment providers (e.g., Stripe, PayPal).
* Payment confirmation within **3 seconds**.
* Support multiple currencies.

---

## 📩 5. Notification System

### 🎯 Objective

Notify users (Guests, Hosts) of key events like booking confirmation, cancellations, and payment updates via email and in-app alerts.

### 🔌 API Endpoints

#### `POST /api/notifications/send`

* **Auth**: Internal (triggered via service)
* **Input**:

  ```json
  {
    "recipientId": "user123",
    "type": "booking_confirmation",
    "message": "Your booking has been confirmed!"
  }
  ```

#### `GET /api/notifications`

* **Auth**: Logged-in user
* **Output**:

  ```json
  [
    {
      "id": "notif_001",
      "message": "Your payment was successful.",
      "seen": false,
      "timestamp": "2025-07-01T12:00:00Z"
    }
  ]
  ```

### ⚙️ Performance Criteria

* Email delivery via SendGrid or Mailgun.
* Instant in-app notification rendering.
* Notifications auto-delete after **30 days** (optional).

---

## 🧑‍💼 6. Admin Dashboard

### 🎯 Objective

Allow admins to monitor and manage users, listings, bookings, reviews, and payments for platform integrity and troubleshooting.

### 🔌 API Endpoints

#### `GET /api/admin/users`

* **Auth**: Admin
* **Purpose**: View all registered users
* **Query Params**: Support pagination and search

#### `GET /api/admin/bookings`

* **Auth**: Admin
* **Purpose**: Audit all bookings, statuses, and user associations

#### `DELETE /api/admin/reports/:reportId`

* **Purpose**: Act on reported content (e.g., fraudulent listing)

### ⚙️ Performance Criteria

* Role-based access enforced via middleware.
* Admin routes protected with elevated privilege checks.
* Admin dashboard supports bulk actions and CSV export.

