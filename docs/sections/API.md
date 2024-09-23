---
title: API
tags: [Internal Documentation]

---

## Notes on CORS

Without CORS enabled, you can indeed not send a query from the query tool to the API in any (of the tested) settings. That is because the query tool and the API run on [different Origins](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#definition_of_an_origin) even locally. All as documented. 

The next question is what we should do about this and what the risks are. My understanding of the former is that the safest bet in this order would be:
- don't allow CORS
- allow CORS only for a-priori trusted origins
- allow CORS but don't return `*` in the [CORS request response headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#the_http_response_headers). 

My understanding of the latter part isn't as clear yet, but essentially it seems to be that the risk is "not very big" for our current public API, but "potentially severe" for internal APIs. The risk is that a trusted origin (query.neurobagel.org) loads some script from an untrusted place (evil.org) and that script now does silly things against our API and is then allowed to read the results. Because our API is meant to be queried by anyone, I think the consequences of this happening would be very limited to none. For an internal API this is obviously very different, because that API expects to only be talked to from inside the network and so an attacking script could gain access to information it is not allowed to see. 

Because we want to build things with the most constrained use case in mind, my conclusion is that CORS has to go in the way we use it now (which to be fair was mostly for prototyping anyway). Now we've also come to like functional web tools, so we'll have to find a solution for how the query tool and the API can still work together. There are a couple.

For a public deployment (like ours on query.neurobagel.org), we have two choices I can see:
1. explicitly only allow the trusted origin (query.neurobagel.org)
2. host the API and the query tool from the same origin. This would mean that
  - we host both by ourselves from our server (no more GH pages for the query tool)
  - we change their URLs to be routes on the same domain (e.g. neurobagel.org/api and neurobagel.org/query)
  - we configure nginx to route these internally to the correct ports

For now I vote for option 1 because it is less involved and still quite reasonable. But we could go to 2. directly.

For an internal deployment (whether dev or intranet) to have the same outcome, we need to instead allow the localhost port the query tool is running on. So we should explicitly allow localhost:3000 if the query tool is hosted on :3000. This needs to be transparently configurable by the user for obvious reasons. It's of course also possible to configure nginx locally to avoid CORS alltogether, but I think that's a pretty high bar for a first deployment - particularly because we don't ship a configured nginx with our stuff.

Oh and although we don't have this yet (but definitely want to): for e2e tests during the CI, we would likely use the same setup as for an internal deployment and expect it to work.

Additional notes:
- none of this will affect direct (e.g. via curl) requests to the API - they will still work
- out of curiosity I'd be interested if `allow CORS but don't return `*` in the header` and how that could be achieved


## Neurobagel-hosted APIs

Server name: `fairmount`  
IP: 206.12.99.17

API/graph database name | graph backend | port | URL
---- | ---- | ---- | ----
open_neuro | GraphDB | 8000 | https://api.neurobagel.org
federation | N/A | 8080 | https://federate.neurobagel.org
indi | GraphDB | 8001 | https://indi.neurobagel.org

Server name: `st-viateur`  
IP: 206.12.89.194

API/graph database name | graph backend | port
---- | ---- | ---- |
mni_ppmi | GraphDB | 8888
mni_qpn | GraphDB | 8080
