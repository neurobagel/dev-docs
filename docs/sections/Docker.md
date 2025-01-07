---
title: Docker
tags: [SysAdmin, Deployment]

---

## Internal port vs host port
In the context of docker compose and within a docker network services talk to each other on their internal port number, and not the port that is exposed to the host.
For example, given a stack with the following config
```
name: api
services:
  api:
    environment:
      NB_API_ALLOWED_ORIGINS: '*'
      NB_API_PORT: "8000"
      NB_GRAPH_ADDRESS: graph
      NB_GRAPH_DB: test_data/query
      NB_GRAPH_PASSWORD: admin
      NB_GRAPH_PORT: "5820"
      NB_GRAPH_USERNAME: admin
      NB_RETURN_AGG: "false"
    image: neurobagel/api:test
    networks:
      default: null
    ports:
    - mode: ingress
      target: 8000
      published: "8888"
      protocol: tcp
  graph:
    image: stardog/stardog:8.2.2-java11-preview
    networks:
      default: null
    ports:
    - mode: ingress
      target: 5820
      published: "5821"
      protocol: tcp
    volumes:
    - type: bind
      source: /home/ubuntu/stardog_root_test
      target: /var/opt/stardog
      bind:
        create_host_path: true
  query:
    environment:
      API_QUERY_URL: http://localhost:8000/
    image: neurobagel/query_tool:latest
    networks:
      default: null
    ports:
    - mode: ingress
      target: 3000
      published: "3000"
      protocol: tcp
networks:
  default:
    name: api_default
```
The api will be sending its requests to the port `5820` and not `5821`.

### Note
This may introduce issues for the API -> graph connection when the running API is not in the same Docker network as the graph, e.g. if you have locally spun up an API using Python but want to communicate with a Dockerized graph endpoint. 
In this scenario, you would likely need to manually specify the _host_ port for the graph container in the query URL, rather than using the _container_ port.

## Setting environment variables

### docker compose
- `environment` instruction for a specific service (`in docker-compose.yml`)
  - Defines variables to be set inside the container (compose config file itself cannot see the variables)
- `env_file` instruction for a specific service (`in docker-compose.yml`) 
  - Only sets variables inside the container (compose config file itself cannot see the variables)
  - Can replace / works without `environment` instruction to set variables in the container
  - Values inside specified env file can be quoted or unquoted, with same result in `docker compose config`
- `--env-file` CLI argument (e.g., `docker compose --env-file custom.env ...`)
  - Only seen by the compose config itself, **UNLESS** the `environment` instruction is also used (in which case, variables that are defined under `environment` which also have a value inside the specified env file are passed to and can be used inside the container)
  - Essentially just for specifying a custom path for the `.env` file, and works the same as having a file named `.env` (in which case the `--env-file` argument is not needed)
  - Does not care about quotes around variable values in the file

Some important references:
- https://docs.docker.com/compose/environment-variables/set-environment-variables/
- https://docs.docker.com/compose/compose-file/05-services/ (see sections on `env_file` and `environment`)
- https://docs.docker.com/compose/environment-variables/env-file/
- https://docs.docker.com/compose/environment-variables/envvars-precedence/

### docker CLI (docker run)
All three options below can set variables **inside** the container.

- `ENV` instruction in dockerfile
  - Should work the same as `--env` / `-e` command-line argument (e.g., `docker run -e MYVAR=MYVALUE`)
  - Quotes are needed for variable values with spaces
  - Not required for an environment variable to be available to code in the container, such as if one of the below two options are used
- `--env-file` CLI argument
  - Interprets value exactly as is inside specified file (including any quotes), does not do any parameter expansion for values inside the file (i.e., for `VAR=VALUE`, the VALUE is passed in exactly as is, as a literal)
    - See:
       - https://github.com/docker/for-linux/issues/1208
       - https://github.com/docker/cli/issues/4347#issuecomment-1590210864
  - **IMPORTANT**: THIS IS (FOR SOME REASON) **NOT** THE SAME FUNCTIONALLY AS THE `--env-file` CLI ARGUMENT FOR DOCKER COMPOSE!
    - See this commment https://github.com/docker/cli/issues/4347#issuecomment-1590214094 from a Docker maintainer
  - Unfortunately these caveats are not very well documented 😢
- `--env`/`-e` CLI argument
  - Quotes are needed for variable values with spaces, will **override** the same variable set via `ENV` instruction in the dockerfile

Some important references:
- https://docs.docker.com/engine/reference/commandline/run/#env
- https://docs.docker.com/engine/reference/builder/#env
- https://stackoverflow.com/a/63640896 (this answer mostly explains `ARG`/`ENV` but also covers some info on both `docker run` and `docker compose`)
  - a summary of https://vsupalov.com/docker-arg-env-variable-guide/

## Using `secrets` in Docker Compose

The key point to note when using secrets is that *each* secret that a container (i.e., service in the Compose file) is granted access to gets mounted at a special path inside the container: `/run/secrets/<secret_name>`

That is, the value of each secret ends up as the _contents_ of a separate file under `/run/secrets/` inside the container.



You can define a secret one of two ways:
- By setting it as the value of environment variable that is accessible to the **Compose file itself**
    - So, any way that you can provide/expose environment variables to the Compose file, you should be able to include the environment variable for the secret (see [Setting environment variables](#Setting-environment-variables) section)
      - including via a `.env` file
    
      Example:
      ```bash
      # .env file
      MY_SECRET=bageley
      ...
      ```
      
      ```yaml
      # docker-compose.yml
      ...
      secrets:
        my_secret:
          environment: "MY_SECRET"
      ```
      Inside a container that has access to `my_secret`, `cat /run/secrets/my_secret` will return `bageley`.
      
      If `MY_SECRET` is not found in the environment (or unset ? - need to check), docker compose will give an error.
      
      **NOTE:** In order to ensure the `MY_SECRET` value does not end up exposed in e.g., the docker compose config, if a container needs to use *other* variables defined in the `.env` file, each needed variable should be explicitly specified under the `environment` attribute in docker-compose.yml, rather than using the `env_file` attribute (because it will auto-include *all* the variables in `.env` as environment variables, including the secret).
    
- By making the value of the secret the contents of a file, which you then provide to the `secrets` attribute
  - Example: see https://docs.docker.com/compose/use-secrets/#advanced

Note that for both these methods:
- The secret is **not** automatically set as an environment variable inside the container

References:
- https://docs.docker.com/compose/use-secrets/
- https://docs.docker.com/compose/compose-file/09-secrets/


## Troubleshooting Docker/Docker Compose recipes

Mostly encountered when trying to deploy our tools on the internal BIC node (where we don't have admin permissions), which involves a network FS from a different machine.

### Running Docker container as non-root user

Sometimes, probably depending on the permissions of a user on the machine they are using to deploy Neurobagel, Docker can be run as the `nobody` user (even though inside the container the user is `root`, by default). This means when the container tries to write to a mounted directory on the host, it might not have permissions to do so (unless the directory has `rwx` permissions for ALL users, which usually isn't the desired setup).

> You can verify what user is being used to write to the host FS by running an interactive bash shell inside the container that uses the volume mount, and trying to write to a mounted test directory with fully open access permissions, e.g.:
> ```bash
> docker run -it -v /data/origami/mount_test:/opt/graphdb/home --entrypoint /bin/bash ontotext/graphdb:10.3.1 -c "touch /opt/graphdb/home/hellofromcontainer.txt"
> ```
> Then, you can `cd` into the mounted directory on the host and view the file owner/group.

One workaround is to explicitly run the container as a non-root user, using the [`--user` flag in `docker run`](https://docs.docker.com/engine/reference/run/#user), or the [`user` instruction in docker-compose.yml](https://docs.docker.com/compose/compose-file/05-services/#user):

e.g.,
1. Add following to the relevant service in `docker-compose.yml`:
```bash
user: ${CURRENT_UID}
````
2. Start the Compose stack with 
```bash
CURRENT_UID=$(id -u):$(id -g) docker compose up -d
```

Useful commands for troubleshooting:
```bash
id $USER  # shows uid and all gid and names for current user
id -gn  # gets name of primary group

# get id of user and primary group
id -g  # not all shells set $GID
id -u  # same as $UID
```

References:

- https://medium.com/redbubble/running-a-docker-container-as-a-non-root-user-7d2e00f8ee15
- https://docs.docker.com/compose/compose-file/05-services/#user
- https://docs.docker.com/engine/reference/run/#user
- https://dev.to/izackv/running-a-docker-container-with-a-custom-non-root-user-syncing-host-and-container-permissions-26mb
- https://askubuntu.com/questions/1201352/what-environment-variable-will-report-the-users-primary-group

### Specifying specific subnet for Docker networks

Unless configured to do otherwise, the Docker engine by default will choose a subnet for each Compose network at creation time - this is problematic when an existing subnet needs to be reserved for use on a machine, e.g. for a specific network FS. When this is the case, the network FS becomes unreachable if a conflicting subnet is used by a newly launched Docker network. (Note: By default, each Docker Compose network gets assigned its own subnet.)

There are a few ways to change which subnet is used by a Docker network or Docker container:

- edit `/etc/docker/daemon.json` (see https://github.com/docker/compose/issues/4336#issuecomment-457326123)
  - you can set `"default-address-pools"`, which will put containers using both docker and docker compose into your desired subnet
  - setting `"bip"` and `"fixed-cidr"` only seems to affect Docker containers launched not as part of a Compose recipe, which use the default bridge network
- specify a network subnet in each docker-compose.yml, following https://docs.docker.com/compose/compose-file/06-networks/#ipam
  - option 1: specify to use the _same_ network in each docker-compose.yml - see example [here](https://stackoverflow.com/questions/38088279/communication-between-multiple-docker-compose-projects) (this is != specifying just the same `subnet` in each docker-compose.yml, which can cause the following issue)
    ```bash
    [+] Running 1/0
    ✘ Network bic_federation_fednet  Error                                                                                  0.0s
    failed to create network bic_federation_fednet: Error response from daemon: Pool overlaps with other one on this address space
    ```
  - option 2: use a custom network per docker-compose.yml, but specify non-conflicting `subnet`s (either /16 or /24)
    - specifying the same `subnet`, but non-conflicting `ip_range` doesn't seem to be sufficient:
    ```bash
    Error response from daemon: cannot create network c800e759f35a326a36f8beac2aa60fc5032a8cc92ede2210a92ba8f4845e9c5f (br-c800e759f35a): conflicts with network 6873e8489de40af4d4a9f7635145fd3ea9753fb4c03f6eafbc6e5fa8fcdde791 (br-6873e8489de4): networks have overlapping IPv4
    ```
    - specifying just `ip_range` also seems to not work?
    ```bash
    failed to create network bic_federation_fednet: Error response from daemon: Invalid subnet  : invalid CIDR address:
    ```

Resources:
- https://docs.docker.com/config/daemon/ipv6/#dynamic-ipv6-subnet-allocation
- https://docs.docker.com/compose/compose-file/06-networks/#ipam
- https://forums.docker.com/t/docker-default-address-pool-customization-question/112969
- https://docs.docker.com/network/network-tutorial-standalone/#use-user-defined-bridge-networks
- https://stackoverflow.com/questions/38088279/communication-between-multiple-docker-compose-projects