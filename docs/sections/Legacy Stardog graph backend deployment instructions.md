---
title: Legacy Stardog graph backend deployment instructions

---

:warning: The below instructions are the old mkdocs version of the Neurobagel node setup instructions using stardog, which are now deprecated. :warning: 

## Setup for the first run

To interact with your graph backend, you have two general options:

1. Send HTTP requests to the HTTP REST endpoints of the Stardog graph backend (e.g. with `curl`). See [https://stardog-union.github.io/http-docs/](https://stardog-union.github.io/http-docs/) for a full reference of Stardog API endpoints
2. Use the free Stardog-Studio web app. See the [Stardog documentation](https://docs.stardog.com/stardog-applications/dockerized_access#stardog-studio) for instructions to deploy Stardog-Studio as a Docker container.


!!! info 
    Stardog-Studio is the most accessible way 
    of manually interacting with a Stardog instance. 
    Here we will focus instead on using the HTTP API for configuration,
    as this allows programmatic access.
    All of these steps can also be achieved via Stardog-Studio manually.
    Please refer to the 
    [official docs](https://docs.stardog.com/stardog-applications/studio/) to learn how.
    
### Set the database admin password

When you first launch the graph server, a default `admin` user with superuser privilege will automatically be created for you. 
This `admin` user is meant to create other database users and modify their permissions.

You should first change the password of the database `admin`:

```bash
curl -X PUT -i -u "admin:admin" http://localhost:5820/admin/users/admin/pwd \
--data '{"password": "NewAdminPassword"}'
```

### Create a new database user

We do not recommend using `admin` for normal read and write operations, instead we can create a regular database user.

The `.env` file created as part of the `docker compose` setup instructions
declares the `NB_GRAPH_USERNAME` and `NB_GRAPH_PASSWORD` for the database user.
The Neurobagel API will send requests to the graph using these credentials.
When you launch the RDF store for the first time, 
we have to create a new database user:

```bash
curl -X POST -i -u "admin:NewAdminPassword" http://localhost:5820/admin/users \
-H 'Content-Type: application/json' \
--data '{
    "username": "DBUSER",
    "password": [
        "DBPASSWORD"
    ]
}'
```

Confirm that the new user exists:

```bash
curl -u "admin:NewAdminPassword" http://localhost:5820/admin/users
```

!!! note
    Make sure to use the exact `NB_GRAPH_USERNAME` and `NB_GRAPH_PASSWORD` you
    defined in the `.env` file when creating the new database user.
    Otherwise the Neurobagel API will not have the correct permission
    to query the graph.
    
### Create new database

When you first launch the graph store,
there are no graph databases.
You have to create a new one to store
your metadata.

If you have defined a custom `NB_GRAPH_DB` name in the `.env` file,
make sure to create a database with a matching name.
By default the Neurobagel API will query a graph database named `test_data`.

```bash
curl -X POST -i -u "admin:NewAdminPassword" http://localhost:5820/admin/databases \
--form 'root="{\"dbname\":\"test_data\"}"'
```

### Grant database permissions to user

Now we need to give our new database user read and write permission for 
this database:

```bash
curl -X PUT -i -u "admin:NewAdminPassword" http://localhost:5820/admin/permissions/user/DBUSER \
-H 'Content-Type: application/json' \
--data '{
    "action": "ALL",
    "resource_type": "DB",
    "resource": [
        "test_data"
    ]
}'
```

??? note "Finer permission control is also possible"

    For simplicity's sake, here we give `"ALL"` permission to the new database user.
    The Stardog API provide more fine grained permission control.
    See [the official Stardog API documentation](https://stardog-union.github.io/http-docs/#tag/Permissions/operation/addUserPermission).