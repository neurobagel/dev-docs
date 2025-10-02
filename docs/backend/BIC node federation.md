---
title: BIC node federation
tags: [Deployment]

---

BIC internal node federation
===
Below are some problems uniquely encountered on the `neurobagel-node` VM.

(See also [Docker](/-SrfAKVJTsu4yCV4oxDR4w))

#### `host.docker.internal`
- The f-API (in either a standalone container on the default bridge network, or our `local_federation recipe`) cannot access services on `host.docker.internal`
- Seemingly, only using the external URL for the n-API (NGINX) in `local_nb_nodes.json` works
- `host.docker.internal` successfully resolves to whatever the bridge network IP address is, but services with published ports on the host still can't be reached


#### Reaching VM from inside container on VM
- from inside the `neurobagel-node` VM we can connect to a n-API running on the same machine using the public URL of the n-API (https://api-bic.neurobagel.org)
- **however**: if a request is made from **inside** a docker container on this machine using the public IP or hostname of the machine, it fails
  - `ping` works, but `curl` fails, indicating a firewall/port issue
- this problem only arose after access to the node became restricted using iptables

Our *temporary* solution (just for this machine):
- We manually add the f-API container to same network as the n-API+graph
- We update `local_nb_nodes.json` on `neurobagel-node` to use the container name of n-API as the `ApiURL` for the federation index rather than its public URL

Here are the commands we use for that:

```bash
# docker network connect <n-API-network> <f-API-container>
docker network connect bic_pd_node_nodenet bic_federation-federation-1
```
**Note:** this command needs to be rerun whenever the f-API Compose stack (i.e., the f-API container) has been stopped and restarted.


and long term:
- we want to switch to using one Compose network for all services so that all requests come from inside network

