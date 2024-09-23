---
title: OAuth / OIDC dump

---

OAuth / OIDC dump
===

# Seb brain dump
- OAuth is a protocol that allows a user to give an app (A) specific (scoped) access to data in an app (B) without revealing credentials
    - classic example: budget app (A) wants to see your transaction in your bank (B). Historically (and still today) this (insanely) involves(d) sharing your actual username and password with app (A)
    - OAuth removes the need to share credentials
- OAuth bottom line is:
    - give access to a resource
- OIDC is a layer on top of OAuth
    - because OAuth works so well and identity for things like SSO were hard before
    - OIDC is mainly used so app A can let you log in with e.g. github and then show your beautiful profile picture and some other basic info about you as a user
    - so the "resource" OIDC accesses is your identity
- There are different OIDC flows (implicit, some other thing)
    - A lot of the security relies on the fact that an app that wants to use OIDC with a specific identity provider (e.g. github) first has to register with the identity provider
    - that means, my app goes to github, says "hello, it's me, the app, here is my public key. when I call back later to ask about Sebs account, this will prove it's really me asking"
    - When Seb logs into the identity provider (IP) there is a specific prompt "Do you want to give app A access to the following scopes?" (e.g. email, name, avatar ...). If Seb says yes, there is a redirect URL that includes a special code.
    - The App can now parse the code from the redirect URL and then go to the IP and say: "Hey, Seb just said I could look at his avatar and email. Here is a code to prove that. Can I please get an identity token in exchange". And then the IP gives the token back which includes a bunch of other things, but mainly says "Yes, I, the Identity Provider, confirm that this is really Seb you just talked to"
    - Finally, the App can exchange the identity token for more detailed user info about Seb at a identity resource API from the identity provider


Final thing:
- because identity tokens are issued by the IP on behalf of a user, for the consumption of a specific app, things (may) get tricky when we introduce our federated network. 
- Because the app in our case would be e.g. the query tool, but each node in the network would have to validate the identity token it receives with the federated query. 
- So probably we will have to look into something called Token Exchange (https://oauth.net/2/token-exchange/).
- I don't get that whole thing fully yet - but my understanding is that Neurobagel would get the initial OIDC token, and would then exchange the token for a new token for each node in the network. 
- So from the perspective of a Neurobagel node that receives a federated query, the identity token would be asserted by Neurobagel, and not by the identity provider. 
- If that is the case (e.g. Neurobagel would have to be an intermediary identity provider) that would not be great. Ideally we can find a way how a user goes through one OIDC flow and then each node can validate the token


Take a look at some of the resources I found useful. Most useful probably is the GitHub OIDC flow tutorial. I think the tutorial is not in python, but should be reasonably easy to translate.

### 2024-07-02
Implementation thoughts:
- Step 1: OIDC workflow should run in the frontend
    - Should make GH user icon appear in top right corner of query tool (showing context from the OAuth)
- How does query tool talk to federation API differently after logging into GH?
  - Auth scoped to specific application
  - Query tools becomes a registered OAuth app on GH
  - What do we do w/ identity token
  - Step 2: As part of request to f-API, JWT is set to f-API, which then double checks w/ GH 
      - f-API then sends that over to n-API, which does the GH check again
  - issue: every jump needs a token
  - needs to check at every step = zero trust
      - otherwise, everyone needs to essentially trust the f-API instead of a third party app (e.g., GitHub)
- this is called token exchange
- f-API needs to validate who is sending request
- feature flag for dev.

## Resources
- https://www.youtube.com/watch?v=ZV5yTm4pT8g (generally good site)
- https://docs.github.com/en/apps/creating-github-apps/writing-code-for-a-github-app/building-a-login-with-github-button-with-a-github-app
- https://www.youtube.com/watch?v=CPbvxxslDTU
- https://www.youtube.com/watch?v=rTzlF-U9Y6Y
- https://auth0.com/blog/id-token-access-token-what-is-the-difference/

# Alyssa's notes
## More on token exchange
- The app that receives the initial token (?) has the option to impersonate (keep app identity hidden from downstream app and act as end user) or delegate (app is acting on behalf of the client, which has obtained auth. from end user) identity of user
- Tokens include an 'audience value' (who is this intended for) and 'scopes' (what things can you see)

- Generally, when token exchange is involved there is a central _authorization server_ that becomes the single point of trust. This is typically not GH, which primarily acts as an _identity provider_ (encompasses an _authentication server_) and may not support token exchange natively
    - an authorization server handles the authorization aspects of token issuance, and can (but does not always) also handle token exchange
    - GH may not act as a full-fledged authorization server in the OAuth 2.0 sense
        - Supported OAuth grant types in GH are mentioned here: https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps
        - GH doesn't seem to support RFC 8693, which defines the OAuth 2.0 Token Exchange protocol
    - GH’s access tokens typically grant access to resources within GH itself (e.g., repositories, issues) rather than acting as tokens that authorize access to third-party services or APIs unrelated to GH
        - GH's access tokens are scoped to its own APIs and are not intended to manage access to arbitrary external resources / APIs
- For token exhange, can use a service like Keycloak or Auth0
    - will allow us to use GH for initial user authentication and then manage service-specific tokens through a central authorization server

- w/o token exchange:
    - each downstream service must trust token issued by authentication server (GH), and must validate the received token against GH APIs
    - usually means the token's scope needs to be broader
- w/ token exchange:
    - minimizes permissions granted to any one scope (each service requests a new token w/ tailored scopes + permissions)
    - need to trust a central authorization server that supports token exchange, like Keycloak, to handle the token issuance

Resources:
- https://sagarag.medium.com/oauth2-token-exchange-in-practice-5a12a6d2e0d
- https://dev.to/fuegoio/demystifying-authentication-with-fastapi-and-a-frontend-26f5
- https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps
- https://medium.com/keycloak/github-as-identity-provider-in-keyclaok-dca95a9d80ca
- https://yasasramanayaka.medium.com/oauth-2-0-token-exchange-flow-78fadf23e1fc

### Implicit vs. authorization code schemes/flows
- **implicit** scheme: frontend receives access token from IdP and uses it to access OAuth resources, and then gives token to backend API to secure connection b/w frontend & backend
- **authorization code** scheme: backend is the one that first receives the access token, & is in charge of accessing resources
    - **[more secure (and generally recommended)](https://tools.ietf.org/html/draft-ietf-oauth-security-topics-12)** b/c frontend doesn't have to give token to backend (although in federated authentication this kind of may be needed anyways)
- another flow is the **password** flow, which is the one used in FastAPI tutorials

Resources:
- https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps#web-application-flow
- https://github.com/tiangolo/fastapi/discussions/9141

### Using external OAuth providers
- GH
    - **not an OIDC provider** -> doesn't provide ID token
    - does not support implicit flow: https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps
- ORCiD
    - OIDC provider with nice [documentation](https://github.com/ORCID/orcid-auth-widget?tab=readme-ov-file), but we need to go through an app registration process
        - for ORCiD members (member API): https://info.orcid.org/documentation/integration-guide/registering-a-member-api-client/
            - think this is more for use for member **organizations** (e.g., McGill?)
        - for non-members (public API) https://info.orcid.org/documentation/integration-guide/registering-a-public-api-client/
    - both the member API & public API also have a corresponding sandbox API version, for which you can apply for credentials for your application. The only extra step is that this needs to be done through a [sandbox ORCiD account](https://sandbox.orcid.org/signin), using a mailinator.com email
- Google
    - OIDC provider
    - General info: https://developers.google.com/identity/openid-connect/openid-connect
    - For example implementation using authorization code flow: https://github.com/kolitiri/fastapi-oidc-react
    - https://stackoverflow.com/questions/7130648/get-user-info-via-google-api/67913727#67913727
    - Setting up JS origins / redirect origins for Google API client:
        - https://developers.google.com/identity/protocols/oauth2/javascript-implicit-flow#origin-validation
        - https://developers.google.com/identity/protocols/oauth2/web-server#uri-validation

- `client_id` and `client_secret` are important, they are provided by the IdP for the app (frontend?) and can be used to verify a received access token against the IdP (e.g., the access token that the frontend passes to the backend API)

Related FastAPI resources:
- https://github.com/tiangolo/fastapi/discussions/9137
- https://github.com/tiangolo/fastapi/blob/master/fastapi/openapi/models.py#L387-L391

## OAuth vs. OIDC vs. JWT (authentication)
- JWT is a token format, can be used for authentication ("JWT authentication")
  - a way to encode and verify claims for a token
  - not a (OAuth?) protocol / standard, doesn't specify how client _obtains_ the token
  - JWT tokens are signed with a secret key
- OIDC
  - uses access tokens and ID tokens (ID token is a JWT, access token may (?) be a JWT but doesn't have to be)
- OAuth
  - typically deals with only access tokens

Resources:
- https://stackoverflow.com/questions/39909419/what-are-the-main-differences-between-jwt-and-oauth-authentication


## More on token types
- "Access tokens, ID tokens, and self-signed JWTs are all bearer tokens"

Resources:
https://cloud.google.com/docs/authentication/token-types

## Resources for using Google as external IdP
- https://medium.com/@vivekpemawat/enabling-googleauth-for-fast-api-1c39415075ea
- https://prokofyev.ch/implementing-google-login-with-javascript-and-fastapi/
- https://developers.google.com/identity/gsi/web/guides/verify-google-id-token#python

Questions:
- What's the relationship of bearer authentication (see also https://stackoverflow.com/a/77426797), implicit OAuth2 flow, and authorization code OAuth2 flow?
    - Either flow can use "bearer" authentication, I guess?
    - w/o token, it's a "basic" auth scheme (?)

### Questions re: federated auth
- If the n-API and f-API in a local deployment use the same setting for `NB_ENABLE_AUTH`, what happens when the query gets forwarded to other (e.g., public) nodes that **do** require authentication?
  - Maybe 'disabling' means making it optional, so that user can still log in to have access to all nodes, if desired?