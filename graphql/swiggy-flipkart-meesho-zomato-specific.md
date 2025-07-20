
## 🏢 Swiggy/Flipkart/Meesho/Zomato GraphQL Interview Questions (SDE-2)

### 1. GraphQL vs REST for low-bandwidth mobile networks — what would you choose and why?

* GraphQL allows fetching only required fields.
* Ideal for constrained networks where REST might over-fetch.

### 2. How would you use GraphQL to decouple frontend teams from backend service teams?

* Schema-first approach.
* Define GraphQL schema as contract.
* Backend evolves independently as long as schema is respected.

### 3. Describe your caching strategy in a high-traffic food ordering app using GraphQL.

* Cache per-user queries in Redis.
* Cache popular restaurant/product data at CDN level.
* Use Apollo Client local cache for instant UX.

### 4. Have you used GraphQL with Apollo Client in production? What challenges did you face?

* Cache invalidation.
* Schema versioning.
* Managing deeply nested updates.
* Query batching and error tracking.

### 5. How would you enforce role-based access control at field-level in GraphQL schema?

* Use custom directives like `@auth(role: "admin")`.
* Implement directive logic in server middleware.
* Use context to pass user roles and check them inside resolvers.

---

Prepared for Backend SDE-2 Interviews — GraphQL Focused
