flowchart TD
  A[Guest selects property] --> B[Enter booking details: dates, number of guests]
  B --> C[System checks availability]
  C -->|Available| D[Calculate total booking cost]
  C -->|Unavailable| Z[Show error: Property not available]

  D --> E[Guest proceeds to payment]
  E --> F[Process payment via Stripe or PayPal]
  F -->|Success| G[Store booking in database]
  F -->|Failure| Y[Show error: Payment failed]

  G --> H[Send confirmation email and in-app notification]
  H --> I[Booking confirmed]
