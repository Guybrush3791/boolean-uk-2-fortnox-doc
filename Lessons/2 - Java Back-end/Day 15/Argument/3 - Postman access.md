# Postman Access
## Public end-points
To access *public* end-points you can use *Postman* as always

![[Postman public access|1200]]
## Protected end-points
For the protected end-points is a whole different story
### Get OAuth2 Access Token
To make *Postman* able to access protected resources you have to authenticate your-self in advice. To do that select *Authentication* -->  *OAuth2* and setup the connection as follow:

| Key                       | Value                                                                           |
| :------------------------ | :------------------------------------------------------------------------------ |
| Add authorization data to | **Request Headers**                                                             |
| Token name                | `springboot-realm-1`                                                            |
| Grant type                | **Authorization Code (PKCE**)                                                   |
| Callback URL              | `https://oauth.pstmn.io/v1/callback`                                            |
| Auth URL                  | `http://localhost:8080/realms/springboot-realm-1/protocol/openid-connect/auth`  |
| Access token URL          | `http://localhost:8080/realms/springboot-realm-1/protocol/openid-connect/token` |
| Client ID                 | `springboot-realm-1`                                                            |

Insert the data as in the picture
![[Postman OAuth2 Setup|1200]]

Press **Get New Access Token** and log in when Keycloak prompts

![[Postman OAuth2 Login|1200]]

Once the process is done, on top of the *Authorization OAuth2* tab, you can see token in use, expiration date and *Refresh* button

### Access protected data
Once token is properly stored in *Authorization* tab, you can access protected end-points as any other resource.

---

# Links
![[Lessons/2 - Java Back-end/Day 15/__block/Links]]