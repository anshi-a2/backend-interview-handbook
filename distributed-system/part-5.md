# Maintaining a Single Session Token Across Multiple Auth Servers

## Problem Statement

You have **5 authentication servers** behind a **load balancer**. A user’s requests can hit **any server**. You must ensure:

* One **single session token per user**
* Session remains **valid regardless of which server handles the request**
* System is **scalable, fault-tolerant, and secure**

This is a **very common backend interview question** testing understanding of **statelessness, distributed systems, and auth design**.

---

## Why This Is a Problem

If sessions are stored **in-memory** on each auth server:

* Server A creates session → stored locally
* Next request goes to Server B → session not found ❌

This leads to:

* Random logouts
* Broken authentication
* No horizontal scalability

---

## Core Principle: Stateless Auth Servers

> **Auth servers must be stateless**

Session state must live **outside** the servers.

This allows:

* Horizontal scaling
* Easy rolling deployments
* Fault tolerance

---

## Recommended Solution (Industry Standard)

### Use a Shared Session Store (Redis)

```
Client
  ↓ (token)
Load Balancer
  ↓
Auth Server (any)
  ↓
Shared Session Store (Redis)
```

### How It Works

1. User logs in
2. Auth server:

   * Generates a **session token** (UUID / random string)
   * Stores it in Redis:

     ```
     session_token → user_id, expiry, metadata
     ```
3. Token is returned to client (cookie / header)
4. On every request:

   * Any auth server validates token via Redis

---

## Why Redis Is Ideal

* Extremely fast (in-memory)
* TTL support (automatic session expiry)
* Centralized & shared
* Highly available (replication, clustering)

---

## Session Token Design

### Token Characteristics

* Random, unguessable
* No user data inside
* Short-lived (15–60 min typical)

### Storage Example

```
Key: session:abc123
Value: {
  userId: 42,
  roles: [USER],
  createdAt: timestamp
}
TTL: 30 minutes
```

---

## Alternative Approaches (Interview Comparison)

### 1️⃣ Sticky Sessions (NOT Recommended)

Load balancer always routes user to same server.

❌ Problems:

* Breaks on server crash
* Poor scalability
* Hard to rebalance traffic

✅ Mention only to reject it.

---

### 2️⃣ JWT-Based Authentication (Stateless Tokens)

Instead of sessions:

* Server issues **JWT**
* Token contains claims
* No server-side storage

#### Pros

* No Redis needed
* Fully stateless

#### Cons

* Hard to revoke
* Token leakage risk
* Rotation complexity

💡 Common in **microservices**, less common for **banking login sessions**.

---

### 3️⃣ Hybrid: JWT + Redis

* JWT as access token
* Redis stores blacklist / refresh tokens

Used when:

* You need revocation
* Security is critical

---

## Handling Logout

### Redis Session Model

* Delete session key from Redis
* Token becomes invalid immediately

### JWT Model

* Add token ID to blacklist in Redis
* Checked on every request

---

## High Availability Considerations

* Redis replication / cluster
* Graceful fallback if Redis is slow
* Short Redis timeouts

---

## Banking / Financial Systems Perspective

Banks usually:

* ❌ Avoid pure JWT for login sessions
* ✅ Use **server-side sessions** with Redis / DB
* ✅ Enforce **single active session per user**

Example:

* New login invalidates old session
* Prevents account sharing & fraud

---

## Common Interview Follow-ups

### Q: How do you enforce single session per user?

Store mapping:

```
userId → sessionToken
```

On new login:

* Delete old session
* Create new one

---

### Q: What happens if Redis goes down?

* Use Redis cluster
* Fail closed (deny auth)
* Short cache of validated tokens

---

## Model Interview Answer (Concise)

> "Auth servers should be stateless. I’d store session tokens in a shared store like Redis with TTL. Any auth server can validate the token by querying Redis. This avoids sticky sessions, supports horizontal scaling, allows immediate logout, and works well for high-security systems like banking."

---

## Key Takeaways

* Never store sessions in-memory on auth servers
* Prefer Redis-backed sessions for security-critical systems
* Sticky sessions are a trap
* Statelessness = scalability + reliability


* 🧠 Designing auth for microservices
* 🎯 Mock auth system design interview
