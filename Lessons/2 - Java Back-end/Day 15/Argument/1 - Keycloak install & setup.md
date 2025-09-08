# Keycloak install & setup
## SSO Login through Keycloak

![[Keycloak and SSO introduction|1200]]

**1. The initial knock on the door** A user clicks a link or loads a page in an application that is protected. The app immediately notices, *“I don’t know who you are,”* and—rather than asking for credentials itself—politely hands the browser a **302 redirect** pointing to *Keycloak*.

**2. Meeting the gatekeeper** The browser follows that redirect to *Keycloak*’s authentication endpoint. Keycloak now becomes the *single gatekeeper* for *every application in the realm*, so the user only ever needs to identify themselves once here.

**3. Choosing how to prove identity** Keycloak shows a branded login page:
- If the realm offers only local accounts, the page presents the familiar username/password form
- If external identity providers (*Google*, *Microsoft*, a corporate *IdP*) are configured, the page also shows their buttons. The user decides whether to log in with local credentials or to be sent on to an external provider

**4a. External provider detour (optional)** Suppose the user clicks “Continue with Google.” Keycloak now acts as a *broker*: it constructs a signed authentication request and forwards the browser to Google:
1. Google prompts for *credentials*
2. On success, Google sends the browser back to a special Keycloak callback URL carrying a signed proof (an *OIDC code* or *SAML assertion*) that **“this person is guybrush@example.com.”**
3. Keycloak validates that proof, links it to (or creates) the local user, and moves on

**4b. Internal provider** If no external provider is chosen, Keycloak simply validates the username/password against its own user store

**5. Issuing the passport** With identity confirmed, Keycloak creates two things that form the passport that lets *Guybrush* roam among every client application in the realm without re-authenticating:
- A short-lived **ID token** (plus an access token) signed with the realm’s key
- An SSO **session cookie** that remembers, *“this browser is Guybrush and her session began at 09:42.”*

**6. Delivering the passport to the application** Keycloak now redirects the browser back to the application’s previously registered `redirect_uri`, attaching either:
- an **authorization code** (if OpenID Connect) that the app will immediately swap for tokens, or
- a **SAML response** (if the client speaks SAML)

**7. The application stamps the passport** The application contacts *Keycloak*’s token endpoint (for OIDC) or verifies the SAML signature locally. Once the proof checks out, it stores the resulting access token and records its own session for *Guybrush*. From the user’s perspective, the protected page finally loads.

**8. Seamless roaming afterward** As *Guybrush* jumps to other apps in the same realm, those apps repeat only the very first two steps—“*I don’t know you* → *redirect to Keycloak*”. Yet when her browser reaches *Keycloak* a second time, the SSO session cookie says “*already authenticated*.” Keycloak instantly issues fresh tokens and sends her back, creating the feeling of a single, continuous sign-on *across dozens of services*.

**9. Ending the voyage** The SSO session ends when *Guybrush* explicitly *logs out* or when the *session or refresh token expires*. Until then, *Keycloak* remains the silent orchestrator, sparing her from further credential prompts while keeping each application confident in who he/she is.

## Keycloak Installation
*Keycloak* is the *IdP* (Identity Provider) we will host in docker cluster. An *IdP* is the system that actually proves a user’s identity and issues the signed tokens or assertions that other applications rely on. 

### Boot Keycloak docker instance through *Portainer.io*
To start a *Keycloak* instance, login into *portainer.io* webpage at [http://localhost:9000](http://localhost:9000)/[https://localhost:9443](https://localhost:9443), select local environment and open *Containers* page in the left side menu. Create a new container through `Add container` button in top right of the page and insert following data as shown in picture:

| Key          | Value                                                                       |
| ------------ | --------------------------------------------------------------------------- |
| Name         | Keycloak                                                                    |
| Image        | quay.io/keycloak/keycloak:latest                                            |
| Port mapping | 8080 --> 8080                                                               |
| Command      | start-dev --bootstrap-admin-username admin --bootstrap-admin-password admin |

![[Create Keycloak container in Portainer.io|800]]

Once all images are downloaded and container is running check state on *Container* page
![[Keycloak running container|800]]

You can login to the *Keycloak UI* at [http://localhost:8080](http://localhost:8080), default *username* **and** *password*: `admin`
![[keycloak-login-page.png]]

## Keycloak setup
### Realm concept
In Keycloak, a **realm** is an _isolated security domain_—a self-contained container that holds its own users, roles, groups, authentication flows, and client applications. Think of each realm as a tenant: everything defined inside it is invisible to other realms, which makes it ideal for multi-tenant or multi-environment setups (e.g., _dev_, _test_, _prod_).
### Create a dedicate Realm for SpringBoot application
On the left side menu click on *Manage realms* --> *Create realm* and define the name of the realm: `springboot-realm-1`

![[Keycloak create realm|1200]]

### Create Client
Make sure you are on correct realm

![[Keycloack check proper realm|1200]]

Create the new *client* through *Clients* --> *Create client* button and insert following data into the forms

**1. General settings**

| Key       | Value              |
| --------- | ------------------ |
| Client Id | springboot-realm-1 |

**3. Login settings**


| Key                 | Value                                                                                      | Note                                      |
| ------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------- |
| Root URL            | http://localhost:4000                                                                      |                                           |
| Home URL            | http://localhost:4000                                                                      |                                           |
| Valid redirect URIs | http://localhost:4000/login/oauth2/code/my-oidc-client, https://oauth.pstmn.io/v1/callback | The last URI is useful for *Postman only* |
| Web origins         | http://localhost:4000/                                                                     |                                           |
| Admin URL           | http://localhost:4000/                                                                     |                                           |


![[Keycloak create client|1200]]
### Create user
Make sure you are on correct realm

![[Keycloack check proper realm|1200]]

In order to access services we need to create a user to perform the login.
Create user through *Users* --> *Create new user*, define any values you want and *create it*

![[keycloack-create-user.png|1200]]

Once user is created, make sure you setup a password to perform the login: *Users* --> *click on username* --> *Credentials* --> *Set password*

![[Keycloak user password create|1200]]

---

# Links
![[Lessons/2 - Java Back-end/Day 15/__block/Links]]