# Spring security and SSO
## SSO login and IdP

### SSO Login
**Single Sign-On (SSO)** is an authentication pattern that lets a user log in **once** with a trusted identity provider (*IdP*) and then automatically gain access to multiple independent applications or services without re-entering credentials.

#### Why teams add SSO to REST APIs
- **Centralized authentication**  your service delegates login to an *IdP* (e.g., *Keycloak*, *Auth0*, *Okta*), the API never sees the user’s password; it only trusts *cryptographically signed tokens* (often JSON Web Tokens, *JWT*)
- **Stateless security for REST**  REST endpoints are typically stateless, a bearer token issued by the *IdP* is included *in each HTTP request’s Authorization header*, so no server-side session needs to be stored
- **Consistent access control across microservices**  multiple services can validate the same token signature and shared claims, enforcing a unified security policy while remaining loosely coupled
- **Improved user experience**  after authenticating once, the browser or mobile client re-uses the token to call many back-end services (or UI front-ends) without further prompts
- **Simplified compliance & password hygiene**  password complexity, MFA, account lockout, and other policies live in one place —the *IdP*— rather than being duplicated in every project

### Identity Provider or *IdP*
An **Identity Provider** is the system that actually proves a user’s identity and issues the signed tokens or assertions that other applications rely on:
- Maintains the *user database* and *credential checks* (passwords, MFA, account status)
- Generates and signs an *OpenID Connect token* or *SAML assertion* after successful authentication, acting as the authoritative “source of truth” for _who the user is_
- Sends that proof to a **Service Provider / Client** (your protected app), which validates the signature instead of asking the user to log in again

---

# Links
![[Links]]