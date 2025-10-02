---
title: Apache Jena Fuseki Deployment
tags: [Deployment, Infrastructure, Internal Documentation]

---

*draft*

## Quick start
For a very quick start, you can 
1. Pull the image 
```bash
docker pull stain/jena-fuseki
```
2. Create a `shiro.ini` file for authentication below is an example file:
```ini
[users]
bagel = verysecurepassword, admin

[roles]
admin = *

[urls]
/** = authcBasic, roles[admin]
```

3. Start a container by running the following command to map the port and mount the `shiro.ini` file
```bash
docker run -d --name fuseki -p 3030:3030 -v ./shiro.ini:/fuseki/shiro.ini stain/jena-fuseki
```

## Connect with Neurobagel API
Assuming you've already followed the steps in [quick-start](#quick-start) and that you're running all this locally on your machine you can go ahead and create a dataset using the configured authentication 

1. Create `synthetic` dataset
```bash
curl -X POST -u bagel:verysecurepassword --data "dbName=synthetic&dbType=tdb" http://localhost:3030/$/datasets
```

2. Upload `synthetic_example.jsonld` to the `synthetic` dataset
```bash
curl -X POST -u bagel:verysecurepasswor --data-binary @{path/to/synthetic_example.jsonld}.jsonld -H "Content-Type: application/ld+json" http://localhost:3030/synthetic/data?default
```

3. Lastly if you'd like to use the Fuseki server with the Neurobagel API use the `.env` file below
```.env
NB_GRAPH_USERNAME=bagel 
NB_GRAPH_PASSWORD=verysecurepassword
NB_GRAPH_DB=synthetic/query
NB_RETURN_AGG=true
NB_API_TAG=latest
NB_API_ALLOWED_ORIGINS="*"
NB_GRAPH_PORT=3030
NB_GRAPH_ADDRESS=127.0.0.1
```

## Relevant questions:
- Can we run it as a docker image (from default or do you have to make one)? There isn't an official docker image for Fuseki but I found a community-supported image that I used for trying and testing Fuseki. We can use the image available on the docker hub at [stain/jena-fuseki](https://hub.docker.com/r/stain/jena-fuseki/) or use the DOCKERFILE available on [stain's GitHub repo](https://github.com/stain/jena-docker) to make our own image.

- Can we create a new graph/database and push data into it? Yes in Fuseki they refer to them as `datasets`.

To create a dataset:
```bash
curl -X POST -u {username}:{passwod} --data "dbName={datasetname}&dbType=tdb" {graph_address}:3030/$/datasets
```

To push data into it e.g., a `.jsonld` file:
```bash
curl -X POST -u {username}:{passwod} --data-binary @{path/to/file}.jsonld -H "Content-Type: application/ld+json" {graph_address}:3030/{datasetname}/data?default
```
- Can you query it with some silly SPARQL query? Yes, we can use the following command:
```bash
curl -X POST -H "Content-Type: application/sparql-query" -H "Accept: application/sparql-results+json"  --data 'SELECT * WHERE { ?s ?p ?o } LIMIT 10' -u {username}:{passwod}  {graph_address}:3030/{datasetname}/query
```

- Is there a web GUI to run queries? maybe an extension / buddy-project? Yes, the aforementioned docker image includes a web UI that allows for running queries on the dataset and offers a few other useful info on datasets but it's rather simple compared to Stardog studio

- Can we create user credentials/passwords to protect graphs? Yes, but it's a bit different than Stardog and GraphDB. Fuseki requires a `shrio.ini` file for creating users, setting roles, giving permissions, etc. See [shiro docs](https://shiro.apache.org/documentation.html).

