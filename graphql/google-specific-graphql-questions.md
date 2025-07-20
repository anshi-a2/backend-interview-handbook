

## 🏢 Google-Specific GraphQL Interview Questions (SDE-2)

### 1. Would you use GraphQL in a large-scale distributed system at Google? Why or why not?

* Only when tight control over response shape is needed.
* REST + Protocol Buffers preferred for strong typing and efficiency.
* GraphQL used for frontend-heavy UIs with many microservices.

### 2. How would you make GraphQL work with gRPC/microservices?

* GraphQL as frontend gateway.
* Resolvers delegate to internal gRPC services.
* Use BFF (Backend For Frontend) pattern.

### 3. How would you ensure strong typing across a large team using GraphQL?

* Use schema-first approach with strict CI validation.
* Auto-generate TypeScript/Java types from schema.
* Version schemas carefully.

### 4. What scalability issues can occur with GraphQL and how do you mitigate them?

* Overly complex queries: limit depth/complexity.
* Inefficient resolvers: use DataLoader.
* Gateway bottleneck: scale GraphQL gateway horizontally.

### 5. GraphQL Federation vs gRPC Mesh — when would you use each?

* Federation for aggregating GraphQL services.
* gRPC Mesh when working with backend-heavy binary protocols.

