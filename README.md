# RVRB Infrastructure
This repo houses all of the IaC for running RVRB in all environments.

RVRB is built using services hosted in docker containers. We've provided a `docker-compose.yml` file that is used for each of our environments.

The **Host** folder contains an additional `docker-compose.yml` that houses the service ingress controller, [Traefik](https://traefik.io/traefik/) and our container management platform, [Portainer](https://www.portainer.io/).

Traefik will issue self-signed certificates to work on `localhost`, so you'll have to allow the exception in your browser to get past any warnings you are presented with.

Both `docker-compose.yml` files expect a `.env` file located in the same directory as it; You'll see examples included.

## Running locally
To run RVRB locally, follow the below steps;
- clone this repo.
- ensure you have [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/) installed.
- ensure you have [node](https://nodejs.org/en) version 20 installed ([nvm](https://github.com/nvm-sh/nvm) is a great option to install and manage node versions).
- create the docker network `docker network create external_network`.
- run `./cloneRepos.sh`.
- rename the `example.env` file to `.env` and enter values for each variable. [See Application Environment Variables](#application-environment-variables).
- navigate into the **Host** folder.
  - rename the `example.env` file to `.env` and enter values for each variable. [See Host Environment Variables](#host-environment-variables).
  - startup the host stack by running `docker-compose up -d`.
  - in your browser, navigate to https://portainer.localhost.
  - Setup your Admin login credentials.
  - following the [Portainer Initial Setup guide](https://docs.portainer.io/start/install/server/setup), create a new stack using the file upload option - uploading the *docker-compose-local.yml* found in the root directory.
- visit https://rvrb.localhost.


## Host Environment Variables

**COMPOSE_PROJECT_NAME**  
Will set the name of the docker-compose stack created for the host.

**ROOT_DOMAIN**  
Which domain to prepend to all services. Also used for certificate creation.  
If you own your own domain, you can setup an A record pointing to `127.0.0.1` for local development without security warnings and use that - see [the traefik documentation for more details](https://doc.traefik.io/traefik/https/acme/).

**CERTIFICATE_DNS_PROVIDER**  
Which of the traefik support providers you use for you DNS records.

**CERTIFICATE_DNS_RESOLVERS**  
Comma separated list of DNS server IP address and ports to use to check for propogation of any created DNS entry. You can use the DNS of your provider, or simply use `1.1.1.1:53,8.8.8.8:53` for cloudflare / google dns.

**CERTIFICATE_EMAIL**  
The email to use for certificates.

**DREAMHOST_API_KEY**  
You will need to create an entry for the relevant API key that your DNS provider uses - Dreamhost is an example here as it's used by RVRB.

**TRAEFIK_DASHBOARD_USER**  
Username to provide access to this service.

**TRAEFIK_DASHBOARD_PASSWORD**  
Password to provide access to this service.

**DOCKER_INFLUXDB_INIT_MODE**  
Set in the example to `setup` - this makes sure it reads the provisioning files and sets your Influx environment up.

**DOCKER_INFLUXDB_INIT_USERNAME**  
Username to provide access to this service.

**DOCKER_INFLUXDB_INIT_PASSWORD**  
Password to provide access to this service.

**DOCKER_INFLUXDB_INIT_ORG**  
Which name should InfluxDb use for your organization.

**DOCKER_INFLUXDB_INIT_BUCKET**  
Which bucket should InfluxDb setup to store your metrics in.

**DOCKER_INFLUXDB_INIT_ADMIN_TOKEN**  
A string to for InfluxDb to use as the token to allow programatic access to store metrics.

**DOCKER_INFLUXDB_HTTP_HTTPS_ENABLED**  
Set in the example to `true` to allow both secure and insecure access (as access is behind traefik, each request will actually be secured)

**GF_SECURITY_ADMIN_USER**  
Username to provide access to this service.

**GF_SECURITY_ADMIN_PASSWORD**  
Password to provide access to this service.

**PUBLIC_KEY**  
The public key from a locally generated RSA key to use to validate any generated JWTs. To generate a key, use `openssl genrsa -out key.pem 2048; openssl rsa -in key.pem -outform PEM -pubout -out key.pub` and copy the contents of the `.pub` file.  
Line breaks should be replaced with `\n`

**WATCHTOWER_HTTP_API_TOKEN**  
An arbitrary string that Watchtower will expect to receive in the Authorization header in order validate any external requests

## Application Environment Variables
**POSTGRES_DB**  
Name to use for the initial database created as part of the Postgres setup

**POSTGRES_USER**  
Username to provide access to Postgres.

**POSTGRES_PASSWORD**  
Password to provide access to Postgres.

**MONGO_INITDB_ROOT_USERNAME**  
Username to provide access to Mongo.

**MONGO_INITDB_ROOT_PASSWORD**  
Password to provide access to Mongo.


### Additional notes
If you are running on Windows you may need to manually clone the repositories that the `clonseRepos.sh` file is cloning and run an `npm install` in each cloned repository

## Adding a new service

Assuming SERVICE_NAME is the name you've given the new service that you've created,

```
SERVICE_NAME:
  container_name: SERVICE_NAME
  image: rvrb/SERVICE_NAME:latest
```
Internally, the service can be accessed from other services code using SERVICE_NAME as an address.  
If you need the service to be externally accessible, you'll need to add it to the external network and add some lables to tell traefik to own routing.  
Note the example below enforces all access to external services requires a valid JWT token. If you want to add an externally accessible service that is publically available without any authentication, simpliy omit the label `traefik.http.routers.SERVICE_NAME.middlewares=jwt-plugin`
```
SERVICE_NAME:
  container_name: SERVICE_NAME
  image: rvrb/SERVICE_NAME:latest
  networks:
    - external_network
  labels:
    - traefik.enable=true
    - traefik.http.routers.SERVICE_NAME.rule=Host(`SERVICE_NAME.${ROOT_DOMAIN}`)
    - traefik.http.routers.SERVICE_NAME.middlewares=jwt-plugin
    - traefik.http.routers.SERVICE_NAME.entrypoints=websecure
```

# Connecting to DBs manually
## MongoDB
- install mongodsh using `brew install mongosh`
- connect `mongodh -u MONGO_INITDB_ROOT_USERNAME -p MONGO_INITDB_ROOT_PASSWORD`

- install postgres utils using `brew install libpq`
- connect `psql -h localhost -U root -W -d postgres`

- install redis client using  `brew install redis`
- connect  `redis-cli`
