

## 🏢 Meta/Facebook-Specific GraphQL Interview Questions (SDE-2)

### 1. Why did Facebook create GraphQL? What specific problems were they solving?

* REST APIs caused **over-fetching** and **under-fetching**, especially for mobile apps.
* They wanted a more flexible, efficient, and client-driven solution.
* GraphQL allowed fetching only required fields and supported schema evolution without versioning.

### 2. Explain how Relay and connections work in a typical Facebook-style paginated query.

* Relay uses the **Connection Specification**:

```graphql
query {
  friends(first: 5, after: "cursor") {
    edges {
      node { id name }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

* Helps support consistent pagination across types.
* Encourages cursor-based pagination over offset for better performance and UX.

### 3. How would you model a newsfeed in GraphQL with real-time updates and deeply nested relationships?

* Use **Subscription** for real-time updates (e.g., `onNewPost`).
* Model nested relationships like:

```graphql
type Post {
  id: ID!
  author: User!
  content: String!
  comments: [Comment!]!
}
```

* Use **DataLoader** or batching to resolve nested fields.
* Consider **fragments** and **lazy-loading** of fields for mobile.

### 4. How do you handle versioning and schema evolution in long-lived React Native apps?

* Avoid breaking changes; use `@deprecated` for fields.
* Use client-driven queries to decouple from backend changes.
* Use **persisted queries** and **feature flags** to gradually roll out changes.

### 5. Explain how GraphQL fragments help reduce client bundle size and support component co-location.

* **Fragments** enable reusable field selections:

```graphql
fragment userInfo on User {
  id
  name
  profilePicture
}
```

* Each UI component defines its own data needs.
* Reduces duplication and bundle size by only fetching required fields.
* Promotes modularity and scalability in frontend code.

---

Prepared for Backend SDE-2 Interviews — GraphQL Focused
