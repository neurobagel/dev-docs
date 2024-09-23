---
title: Keycloak

---

## Keycloak

### Terminology
- "Realm": a set of users, credentials, roles, groups that are managed in independently
  - each user belongs to & logs into a realm
  - there's a "Master" realm, which is not recommended for apps
      - only for managing other realms
  - apps are secured under a specific realm

### Features
- supports out-of-the-box IdPs incl. Google, GitHub, LinkedIn, etc.
  - also flexible enough to support integration of any OIDC or SAML 2.0 provider
- After a Keycloak realm has been created, Google can be added as an identity provider (using the Google API client ID & secret)
  - A user can then log in to the Keycloak conigured client with Google (as well as a standard login)
- You can also create a client in Keycloak directly (and not use an IdP), under a specific realm (which you have manually created users for ?)


### Steps to automate

From the command line (as part of our deploy recipe?):
- create a new realm & configure the IdP for the realm (add credentials for a Google App)
    - Each node controls their own Keycloak instance
- Create a client for the ~~query tool~~ node API
- Create custom client roles (maybe two to start)
    1. Send query + view aggregated results
        - should be the default if user has authenticated
    3. Send query + view non-aggregated results
- enable token exchange


#### Possible workflow
Query Tool (Client in Keycloak):

- User logs in using Google through Keycloak
- Query tool obtains an access token for the user

Federation API (Service):
  - Query tool sends the user's access token to the Federation API.
  - Federation API performs a token exchange using the access token to get a new token for the node API client.
  - Federation API forwards the new token to the node API

Node API (Client in Keycloak):
- Node API verifies the token received from the Federation API.
- Node API uses the verified token to identify the user and perform role-based access control.
- Admin can then view users who have logged in using the admin console and assign more roles

Resources:
https://www.keycloak.org/docs/latest/server_admin/#assigning-permissions-using-roles-and-groups

### Keycloak token exchange
- Client can:
  - exchange existing Keycloak token for one client for token targeted for another client
    - does this work across realms?
  - exchange Keycloak token for token stored for a linked social provider acct (?)
  - exchange external token from another Keycloak realm to internal token
- Token exchange is a client endpoint

#### Q's
- Should f-API (initiating token exchange) talk to the n-API Keycloak or the query tool Keycloak for the exchange process??

### Docker
- keycloak has a docker image to get us started: https://www.keycloak.org/getting-started/getting-started-docker
- They offer docs on customizing and optimizing the container: https://www.keycloak.org/server/containers

### Backend
- They offer docs on how to "secure an application": https://www.keycloak.org/docs/latest/securing_apps/index.html#planning-for-securing-applications-and-services
- The docs mention multiple grant types or as we're familiar with them oauth flows e.g., authorization code, implicit, etc
- The docs don't mention a Python adaptor for OIDC but found: https://github.com/marcospereirampj/python-keycloak/tree/master

Resources:
- https://medium.com/keycloak/github-as-identity-provider-in-keyclaok-dca95a9d80ca
- https://keycloakthemes.com/blog/how-to-setup-sign-in-with-google-using-keycloak
- https://medium.com/@stefannovak96/signing-in-with-google-with-keycloak-bf5166e93d1e

### Keycloak in React
- https://walkingtreetech.medium.com/a-detailed-guide-to-securing-react-applications-with-keycloak-9434a95b4f4f
- https://medium.com/@aalam-info-solutions-llp/how-to-providing-keycloak-authentication-for-react-application-27cab703a386