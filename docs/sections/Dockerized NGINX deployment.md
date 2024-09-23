---
title: Dockerized NGINX deployment

---

Neurobagel deployment
===

- nginx and config
- SSL certificates
- docker compose management
- differences from the public docker compose recipe
- plausible



Resources:
- https://github.com/nginx-proxy/nginx-proxy/tree/main/docs#path-based-routing


## Auto-configured NGINX

- https://github.com/nginx-proxy/nginx-proxy
    - automatically creates new reverse proxy routes for docker containers or docker compose stacks
    - you only need to provide the proxied URL name as a `ENV` variable
    - e.g. `--env VIRTUAL_HOST=foo.bar.com` to create a route for `foo.bar.com`
    - see also: [detailed docs](https://github.com/nginx-proxy/nginx-proxy?tab=readme-ov-file)
- https://github.com/nginx-proxy/acme-companion
    - is a companion project for `nginx-proxy`
    - it automatically requests SSL certificates from letsencrypt for new docker containers or compose stacks
    - you need to only provide another `ENV` variable
    - e.g. `--env "LETSENCRYPT_HOST=foo.bar.com"` to get an SSL certificate for `foo.bar.com`
    - see also: [detailed docs](https://github.com/nginx-proxy/acme-companion/tree/main/docs)

Custom configuration of NGINX for each domain can be done following these instructions:
- https://github.com/nginx-proxy/nginx-proxy/tree/main/docs#custom-nginx-configuration
  - Configuration directories/files should be created locally and then mounted into the NGINX container in the docker-compose.yml
  - e.g. To increase timeout, following lines added to a `conf.d/my_proxy.conf` file:
    ```bash
    proxy_read_timeout 900;
    proxy_connect_timeout 900;
    proxy_send_timeout 900;
    ```

**HOWTO**:
1. Create a `docker-compose.yml` file that contains both `nginx-proxy` AND `acme-companion`. 
    - e.g. here: https://github.com/neurobagel/internal_deployment/blob/1f3bdbf521c0cbeb4590ea9f5d21c3d64020bbad/docker_nginx.yml
    - be sure to expose the docker socket as a volume!
2. Launch the nginx+acme docker compose stack. Take note of the network name that gets created
3. Launch your production docker containers/compose stacks and 
    - add the NGINX+ACME environment variables for the services that need to be publicly reachable:
        - `VIRTUAL_HOST`
        - `LETSENCRYPT_HOST` (should be same as `VIRTUAL_HOST`)
        - `VIRTUAL_PORT` -> this is the internal port used **inside** the container, not the one exposed to the host!
    - add the publicly reachable services to the NGINX+ACME network. E.g. like so: https://docs.docker.com/compose/networking/#use-a-pre-existing-network
        - If using `docker run`, can specify `--net`
4. Look at the nginx+acme logs to ensure that the routes and SSL certificates are created correctly
5. Ensure that the subdomain has been correctly configured in Cloudflare