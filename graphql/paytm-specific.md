

## 🏢 Paytm-Specific GraphQL Interview Questions (SDE-2)

### 1. How would you design a GraphQL schema for Paytm Wallet and UPI?

```graphql
type Wallet { balance: Float!, transactions: [Transaction!]! }
type Transaction { id: ID!, amount: Float!, type: String!, timestamp: String }
type UPI { id: ID!, vpa: String!, linkedBank: BankAccount }
```

### 2. How do you manage high-traffic GraphQL queries in financial applications?

* Query caching.
* Use persisted queries with strict access.
* Monitor/resilience at resolver level.

### 3. Paytm works on mobile-first — how do you optimize GraphQL for mobile?

* Precise queries to reduce over-fetching.
* Batch multiple views in one query.
* Use @defer and pagination.

### 4. Ensuring secure GraphQL access to payment info?

* Enforce OAuth, token scopes.
* Encrypted response headers.
* Mask sensitive fields (like card numbers).

### 5. How to log sensitive queries while ensuring privacy?

* Mask fields during logging (e.g., PAN, phone).
* Use audit trail for resolver calls.

