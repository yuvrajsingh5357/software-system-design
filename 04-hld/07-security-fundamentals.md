# Security Fundamentals in Distributed Systems

Security is a first-class requirement in system design. This chapter covers the authentication protocols, authorization models, network security, encryption, and protection against common OWASP vulnerabilities.

## Table of Contents
- [Authentication: JWT, OAuth 2.0, and OIDC](#authentication-jwt-oauth-20-and-oidc)
- [Authorization: RBAC vs ABAC](#authorization-rbac-vs-abac)
- [Network Security: TLS and CORS](#network-security-tls-and-cors)
- [OWASP Mitigations: XSS, CSRF, and SQL Injection](#owasp-mitigations-xss-csrf-and-sql-injection)
- [Data Encryption: Symmetric vs Asymmetric](#data-encryption-symmetric-vs-asymmetric)
- [Interview Questions](#interview-questions)
- [⚡ Flashcard Summary](#-flashcard-summary)

---

## Authentication: JWT, OAuth 2.0, and OIDC

Authentication answers the question: **"Who are you?"**

### 1. JSON Web Tokens (JWT)
JWT is a stateless authentication mechanism. A JWT has three parts: `Header.Payload.Signature`.
- **Header:** Algorithm used (e.g., HS256, RS256).
- **Payload:** Claims containing user data (e.g., `userId`, `roles`, `expiry`).
- **Signature:** Created by hashing header + payload with a secret key.
- **Security Warning:** Never store sensitive data (like passwords) in a JWT payload because it is only base64 encoded and can be read by anyone.

### 2. OAuth 2.0 vs OIDC (OpenID Connect)
- **OAuth 2.0:** An **authorization** framework. It issues an `access_token` to grant a third-party app permission to act on your behalf (e.g., let Buffer post to your Twitter).
- **OIDC:** An **authentication** layer built on top of OAuth 2.0. It adds an `id_token` (JWT format) to verify the user's identity (e.g., "Sign in with Google").

---

## Authorization: RBAC vs ABAC

Authorization answers the question: **"What are you allowed to do?"**

### 1. Role-Based Access Control (RBAC)
Permits access based on pre-defined roles.
- **Example:** If user has role `ADMIN`, grant access to `/delete`.
- **Pros:** Simple to implement, easy to audit.
- **Cons:** Rigid; hard to handle fine-grained permissions (e.g., "Allow user to delete *only their own* posts").

### 2. Attribute-Based Access Control (ABAC)
Permits access dynamically based on attributes of the user, resource, action, and environment.
- **Example:** "Allow user with attribute `department = HR` to edit resources where `resource.type = EmployeeRecord` between `9 AM and 5 PM`."
- **Pros:** Extremely granular and dynamic.
- **Cons:** High computational overhead to evaluate policies on every request.

---

## Network Security: TLS and CORS

Protecting data in transit and configuring browser policies.

### 1. TLS/HTTPS Handshake
TLS encrypts traffic between client and server:
1. **Client Hello:** Sends cipher suites, TLS version.
2. **Server Hello:** Sends chosen cipher suite + Server Public Certificate.
3. **Key Exchange:** Client verifies certificate with Certificate Authority (CA), generates a pre-master secret, encrypts it with the server's public key, and sends it.
4. **Symmetric Key Derivation:** Both derive a symmetric session key from the secret. Future communications use this symmetric key for speed.

### 2. CORS (Cross-Origin Resource Sharing)
CORS is a browser security mechanism that blocks scripts from one origin (domain) from reading resources from another origin.
- If Origin A makes a `POST` request to Origin B, the browser first sends a **Preflight OPTIONS** request to check B's headers.
- Origin B must respond with headers like `Access-Control-Allow-Origin: https://domain-a.com` to allow the request.

---

## OWASP Mitigations: XSS, CSRF, and SQL Injection

Mitigating the most common web application vulnerabilities.

### 1. SQL Injection (SQLi)
- **Mechanism:** Attackers inject malicious SQL commands into input fields, manipulating backend database queries.
- **Mitigation:** Use **Parameterized Queries** (Prepared Statements) in ORM/JDBC. This treats user input strictly as parameters, never executable code.

### 2. XSS (Cross-Site Scripting)
- **Mechanism:** Injecting malicious JavaScript into web pages viewed by other users.
- **Mitigation:** HTML-encode all dynamic output (contextual escaping) and implement a strict **Content Security Policy (CSP)** header to restrict script source domains.

### 3. CSRF (Cross-Site Request Forgery)
- **Mechanism:** Trick an authenticated user's browser into executing a state-changing request (e.g., submit money transfer form on another site using cookies).
- **Mitigation:** Use **CSRF Tokens** (unique, random tokens verified on state-changing requests) or set cookies to `SameSite=Strict`.

---

## Data Encryption: Symmetric vs Asymmetric

| Dimension | Symmetric Encryption | Asymmetric Encryption |
|-----------|----------------------|-----------------------|
| **Keys** | One shared secret key | Key Pair (Public & Private) |
| **Speed** | Extremely Fast | Slow (high CPU cost) |
| **Usage** | Encrypting bulk data (databases, disks) | Key exchanges, digital signatures, SSL handshake |
| **Examples** | AES-256, ChaCha20 | RSA, ECC (Elliptic Curve) |

*Hashing vs Encryption:* Hashing is one-way (irreversible; e.g., bcrypt for passwords). Encryption is two-way (can be decrypted using the key).

---

## Interview Questions

### Q: Why do we use symmetric encryption after asymmetric exchange in SSL?
Asymmetric encryption (using RSA/ECC key pairs) is extremely resource-intensive. If we used it to encrypt every packet, the server's CPU would bottleneck. Instead, we use asymmetric encryption once during the handshake to exchange a temporary secret key safely, then switch to fast symmetric encryption (like AES-256) for bulk payload encryption.

### Q: How do you secure JWT tokens against theft?
1. Store tokens in HTTP-only, Secure, SameSite=Strict cookies to block XSS and CSRF access.
2. Keep JWT lifespan short (e.g., 15 minutes) and issue a longer-lived `refresh_token` stored securely on the database to rotate access.
3. Use asymmetric signing (RS256) so only authentication service needs access to the private key, while other services verify with public key.

---

## ⚡ Flashcard Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | JWT parts | Header (algo), Payload (data), Signature (validation) |
| 2 | OAuth 2.0 vs OIDC | OAuth: Authorization (Access Token); OIDC: Authentication (ID Token) |
| 3 | RBAC vs ABAC | RBAC: based on roles; ABAC: dynamic attributes (user, environment) |
| 4 | Symmetric Encryption | Single key encrypts/decrypts; fast (AES-256) |
| 5 | Asymmetric Encryption | Public key encrypts, private key decrypts; slow (RSA/ECC) |
| 6 | SQL Injection fix | Parameterized queries (Prepared Statements) |
| 7 | CSRF mitigation | CSRF tokens, SameSite=Strict cookies |
| 8 | XSS mitigation | Output HTML-encoding, Content Security Policy (CSP) |
| 9 | Password storage | Hash with cryptographic salt/pepper using bcrypt or Argon2 |
| 10 | SSL Handshake switch | Asymmetric key exchange → Symmetric session encryption |

---

*Next: [Observability and Monitoring](08-observability-and-monitoring.md)*
