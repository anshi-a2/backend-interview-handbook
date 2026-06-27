# 06_Spring_Security_JWT_Authentication_Internals.md

> **Goal**
>
> Understand **how Spring Security works internally**, from the moment a request enters your application until authentication and authorization are completed.
>
> This chapter covers **Spring Security architecture, Filter Chain, Authentication, Authorization, JWT, OAuth2 basics, Session vs Stateless Authentication, CSRF, CORS, Password Encoding, and production best practices.**
>
> This is one of the **most frequently asked topics in SDE-2/Senior Backend interviews**.

---

# Table of Contents

1. Why Spring Security?
2. Authentication vs Authorization
3. Spring Security Architecture
4. Security Filter Chain
5. Complete Authentication Flow
6. Username & Password Authentication
7. UserDetailsService
8. AuthenticationManager
9. AuthenticationProvider
10. Password Encoding (BCrypt)
11. SecurityContext
12. JWT Authentication
13. JWT Request Flow
14. Session vs Stateless Authentication
15. OAuth2 Basics
16. CORS
17. CSRF
18. Method-Level Security
19. Common Production Issues
20. Best Practices
21. Interview Summary

---

# 1. Why Spring Security?

Without security

```
Internet

↓

Application

↓

Database
```

Anyone can access your APIs.

Spring Security provides

- Authentication
- Authorization
- Password Encryption
- Session Management
- JWT Support
- OAuth2 Integration
- CSRF Protection
- Security Filters

---

# 2. Authentication vs Authorization

Very common interview question.

### Authentication

```
Who are you?
```

Example

```
Username

Password

↓

Verify Identity
```

---

### Authorization

```
What are you allowed to do?
```

Example

```
ADMIN

↓

Delete User

USER

↓

Cannot Delete User
```

---

# 3. Spring Security Request Flow

Complete request flow

```
HTTP Request

↓

Tomcat

↓

Security Filter Chain

↓

Authentication

↓

Authorization

↓

DispatcherServlet

↓

Controller

↓

Service

↓

Repository

↓

Response
```

Notice

Security executes **before** the request reaches your Controller.

---

# 4. Security Filter Chain

Spring Security is built on **Servlet Filters**.

Flow

```
Request

↓

Filter 1

↓

Filter 2

↓

Filter 3

↓

Authentication Filter

↓

Authorization Filter

↓

Controller
```

Common filters

- SecurityContextHolderFilter
- UsernamePasswordAuthenticationFilter
- BearerTokenAuthenticationFilter
- AnonymousAuthenticationFilter
- ExceptionTranslationFilter
- AuthorizationFilter

The exact chain depends on your configuration.

---

# 5. Complete Login Flow

Suppose user sends

```
POST /login
```

with

```
username

password
```

Internally

```
Request

↓

Security Filter

↓

AuthenticationManager

↓

AuthenticationProvider

↓

UserDetailsService

↓

Database

↓

BCrypt Verification

↓

Authentication Success

↓

JWT Generated

↓

Response
```

---

# 6. UserDetailsService

Spring does not know

how users are stored.

It delegates this responsibility.

Example

```java
public interface UserDetailsService {

    UserDetails loadUserByUsername(String username);

}
```

Responsibilities

- Find user
- Load password
- Load roles
- Return UserDetails

---

# 7. AuthenticationManager

AuthenticationManager

coordinates authentication.

Flow

```
Username

Password

↓

AuthenticationManager

↓

AuthenticationProvider
```

It delegates actual verification.

---

# 8. AuthenticationProvider

Responsibilities

- Validate credentials
- Compare password
- Load authorities
- Create Authentication object

Flow

```
Authentication Request

↓

Load User

↓

Compare Password

↓

Success?

↓

Authenticated Token
```

---

# 9. Password Encoding

Never store plain-text passwords.

Bad

```
password123
```

Good

```
$2a$10...
```

Spring uses

```
BCryptPasswordEncoder
```

During login

```
Raw Password

↓

BCrypt

↓

Compare Hash

↓

Authenticated
```

Benefits

- Salted hashes
- Slow hashing algorithm
- Resistant to rainbow table attacks

---

# 10. SecurityContext

After successful authentication

Spring stores user information in

```
SecurityContext
```

Flow

```
Authentication

↓

SecurityContext

↓

Current User Available
```

Access anywhere

```java
SecurityContextHolder.getContext()
```

---

# 11. JWT (JSON Web Token)

JWT is used for

```
Stateless Authentication
```

Structure

```
Header

.

Payload

.

Signature
```

Example

```
xxxxx.yyyyy.zzzzz
```

Payload may contain

```
User ID

Username

Roles

Expiration
```

---

# 12. JWT Authentication Flow

Login

```
Username

Password

↓

Authentication

↓

JWT Generated

↓

Client Stores Token
```

Future Requests

```
Authorization

Bearer JWT

↓

Security Filter

↓

Validate JWT

↓

Create Authentication

↓

SecurityContext

↓

Controller
```

No database lookup is required if the token is self-contained and trusted.

---

# 13. JWT Request Lifecycle

```
Client

↓

Authorization Header

↓

Bearer Token

↓

JWT Filter

↓

Verify Signature

↓

Extract Claims

↓

Create Authentication

↓

SecurityContext

↓

Controller
```

If token invalid

↓

Return

```
401 Unauthorized
```

---

# 14. Session Authentication

Traditional approach

```
Login

↓

Session Created

↓

Session ID

↓

Browser Cookie

↓

Server Validates Session
```

Requires server-side session storage.

---

# 15. Stateless Authentication

Modern microservices prefer

```
JWT
```

Flow

```
Login

↓

JWT

↓

Client Stores Token

↓

Every Request Sends JWT

↓

Validate

↓

Controller
```

Server stores no session.

---

# Session vs JWT

| Session | JWT |
|----------|-----|
| Server stores session | Client stores token |
| Stateful | Stateless |
| Cookie-based | Authorization header |
| Easy logout | Requires token expiration or revocation strategy |

---

# 16. OAuth2 Basics

OAuth2 is

```
Authorization Framework
```

Example

```
Login with Google
```

Flow

```
Client

↓

Google Login

↓

Authorization Code

↓

Access Token

↓

Application
```

OAuth2 is **not authentication by itself**; it is an authorization framework. Authentication is commonly provided through **OpenID Connect (OIDC)** on top of OAuth2.

---

# 17. CORS

Cross-Origin Resource Sharing.

Problem

```
Frontend

localhost:3000

↓

Backend

localhost:8080
```

Different origins.

Browser blocks request.

Solution

```
Access-Control-Allow-Origin
```

Spring provides

```
CorsConfiguration
```

---

# 18. CSRF

Cross-Site Request Forgery.

Example

User logged into

```
Bank Website
```

Attacker tricks browser

↓

Transfer Money

Without user's intention.

Spring Security enables CSRF protection by default for browser-based applications using sessions.

For stateless REST APIs secured with JWT,

CSRF is often disabled because the browser does not automatically attach Authorization headers.

---

# 19. Method-Level Security

Protect individual methods.

Example

```java
@PreAuthorize("hasRole('ADMIN')")
```

Flow

```
Method Call

↓

Check Role

↓

Allowed?

↓

Execute
```

Useful when different users can access different operations.

---

# 20. Security Filter Chain Diagram

```
HTTP Request

↓

Cors Filter

↓

CSRF Filter

↓

Authentication Filter

↓

JWT Filter

↓

Authorization Filter

↓

DispatcherServlet

↓

Controller
```

The exact order depends on your Spring Security configuration.

---

# 21. Common Production Issues

---

## 401 Unauthorized

Meaning

User is not authenticated.

Possible reasons

- Missing JWT
- Invalid JWT
- Expired JWT
- Incorrect credentials

---

## 403 Forbidden

Meaning

User is authenticated

but

does not have permission.

Usually

Role mismatch.

---

## Password Doesn't Match

Check

```
BCrypt

↓

PasswordEncoder
```

Never compare raw passwords.

---

## CORS Error

Browser says

```
Blocked by CORS
```

Configure

```
Allowed Origins

Allowed Methods

Allowed Headers
```

---

## JWT Expired

Return

```
401
```

Generate new access token using your chosen refresh-token strategy.

---

## CSRF Token Missing

Occurs mostly with

- Browser forms
- Session authentication

Less common with stateless JWT APIs.

---

# 22. Production Best Practices

✔ Always use HTTPS.

✔ Never store plain-text passwords.

✔ Use BCrypt or another strong password hashing algorithm.

✔ Keep JWT expiration short.

✔ Store sensitive information outside JWT payloads.

✔ Validate JWT signature and expiration on every request.

✔ Use role-based authorization.

✔ Return proper HTTP status codes (`401` vs `403`).

✔ Enable CORS only for trusted origins.

✔ Use OAuth2/OpenID Connect when integrating with external identity providers.

---

# 23. Interview Summary

## Explain Spring Security Request Flow

"When an HTTP request reaches a Spring Boot application, it first passes through the Spring Security Filter Chain. Authentication filters extract credentials or JWT tokens and delegate authentication to the AuthenticationManager. The AuthenticationManager calls an AuthenticationProvider, which loads user details through UserDetailsService and verifies credentials using a PasswordEncoder such as BCrypt. If authentication succeeds, an Authentication object is stored in the SecurityContext. Authorization checks are then performed, and only if access is granted does the request proceed to the DispatcherServlet and finally the controller."

---

## Authentication vs Authorization

- Authentication answers **'Who are you?'**
- Authorization answers **'What are you allowed to do?'**

---

## What is JWT?

"JWT (JSON Web Token) is a compact, signed token used for stateless authentication. After a successful login, the server issues a JWT containing claims such as user identity and roles. On subsequent requests, the client sends the token in the Authorization header, allowing the server to validate it without maintaining session state."

---

## Why BCrypt?

"BCrypt is a password hashing algorithm that automatically salts passwords and is intentionally computationally expensive, making brute-force and rainbow table attacks significantly more difficult."

---

# Key Takeaways

- Spring Security is implemented primarily through a configurable **Security Filter Chain**.
- **Authentication** verifies identity; **Authorization** determines permissions.
- **AuthenticationManager**, **AuthenticationProvider**, and **UserDetailsService** work together to authenticate users.
- **SecurityContext** stores authentication information for the current request.
- **JWT** enables stateless authentication and is widely used in REST APIs and microservices.
- **BCrypt** should always be used for password hashing instead of storing plain-text passwords.
- **CORS** and **CSRF** solve different security problems and should not be confused.
- Secure applications require not only authentication but also proper authorization, encryption, and secure transport using HTTPS.
