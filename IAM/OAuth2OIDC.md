# OAuth2 and OpenID Connect (OIDC)
---


## Introduction
---

### OAuth2 Roles
---

| Component ID | Component Name | Description|
| -----------  | ----------- |---|
| 01 | The client| The end consumer - may be an user with browser or an application itself (API client) |
| 02 | Resource Owner | |
| 03 | Authorization Server | Azure Active Directory, Okta |
| 04 | Resource Server | A Web Application or an API service which has to be protected, so that only authorized users can access it. It is an relying party from Authorization Server. We can develop a SpringBoot application - to expose few API and secure it with Azure Active Directory|

### Confidential Vs Public Clients
---

### Client Profiles
---

### Tokens and Authorization Code
---
| Token ID | Token Name | Description|
| -----------  | ----------- |---|
| 01 | Authorization Code | An intermediary, opaque code returned to an application and used to obtain an access token and optionally a refresh token. Each authorization code is used once. |
| 02 | Access Token  | A JWT Token, A token used by an application to access an API. It represents the application’s authorization to call an API and has an expiration|
| 03 | Refresh Token | An optional token that can be used by an application to request a new access token when a prior access token has expired.|


### OAuth2 Grants
---

| Grant ID | Grant Name | Description|
| -----------  | ----------- |---|
| 01 | Client Credential Grant | |
| 02 | Authorization Code Grant  | |
| 02A | Authorization Code Grant with PKCE | |
| 03 | Implicit Grant
| 04 | Resource Owner Password Credential Grant |


- https://aws.amazon.com/blogs/mobile/understanding-amazon-cognito-user-pool-oauth-2-0-grants/
- https://dzone.com/articles/an-oauth2-grant-selection-decision-tree-for-securi
- https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html
- https://codeburst.io/authorization-code-flow-with-pkce-oauth-in-a-react-application-dcc4e06798df
- https://aws.amazon.com/premiumsupport/knowledge-center/cognito-custom-scopes-api-gateway/
- https://developer.okta.com/blog/2021/05/05/client-credentials-spring-security
- https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/aad/azure-spring-boot-starter-active-directory/aad-resource-server
