# RVRB Infrastructure
This repo houses all of the IaC for running RVRB in all environments.

RVRB is built using services hosted in docker containers. We've provided `docker-compose.yml` files for each of our environments, as demonstrated by the folders in this repo.

The **Host** folder contains an additional `docker-compose.yml` that houses the service ingress controller, [Traefik](https://traefik.io/traefik/) and our container management platform, [Portainer](https://www.portainer.io/).

Each of the `docker-compose.yml` files expects an .env file located in the same directory as it. You'll see an example included in each folder.

## Running locally
To run RVRB locally, follow the below steps;
- ensure you have [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/) installed.
- ensure you have [node](https://nodejs.org/en) version 20 installed ([nvm](https://github.com/nvm-sh/nvm) is a great option to install and manage node versions).
- copy the `docker-compose.yml` and `getRepos.sh` files from the *Local* folder into a directory on your development machine.
- rename the `example.env` file to `.env` and enter values for each variable.
- run `./cloneRepos.sh`.
- run `docker compose up` (or `docker-compose up` depending on your OS).
- visit http://localhost:8080.

### Additional notes
If you are running on Windows you may need to manually clone the repositories that the `clonseRepos.sh` file is cloning and run an `npm install`in each cloned repository
