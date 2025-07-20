

## 🏢 Airbnb-Specific GraphQL Interview Questions (SDE-2)

### 1. How would you model booking availability in GraphQL?

```graphql
type Property {
  id: ID!
  name: String!
  availability(start: String!, end: String!): [DateAvailability!]
}
type DateAvailability {
  date: String!
  isAvailable: Boolean!
}
```

### 2. How to optimize queries that include map-based filters and location?

* Use spatial indexes in resolvers.
* Add custom scalars for GeoJSON or lat/lng.
* Debounce filters client-side.

### 3. Handling reviews and user content moderation in GraphQL?

* Field-level moderation logic in resolvers.
* Use directives like @moderated or middleware.

### 4. Airbnb has rich search — would GraphQL support fuzzy matching and filters?

* Yes, via resolvers connecting to Elasticsearch.
* Accept filter/sort/pagination as input objects.

### 5. Real-time updates for bookings?

* Use GraphQL subscriptions for host-side updates.
* Push messages on booking confirmations/cancellations.

---

Prepared for Backend SDE-2 Interviews — GraphQL Focused
