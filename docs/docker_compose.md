# Docker Compose

Docker Compose (aka docker-compose) is a simplified method to defining and configuring a whole Docker stack of services, allowing for quick configuration of not just single containers, but multiple containers/services, and their interactivity between each other.

While all functionality of docker-compose can be re-created using only a [Dockerfile](./dockerfile.md) and manual commands or scripted shell script, a docker-compose file allows quick and simple configuration through a single yaml file per environment (e.g. dev, test, prod).

Below we will go over the sections of docker-compose and how they're used, with a final completed sample at the end.

Offial documentation on Docker Compose can be found on [Docker's Docs site](https://docs.docker.com/compose/).


## File Naming

Docker Compose by default, will always look for a `docker-compose.yml` file in the directory where it's commands are ran; if a standard file is found, Docker Compose will use that for executing given commands.

This functionality enables users the option of either:

- Define a standard file (`docker-compose.yml`) for a single environment and be Docker Compose ready for all commands

- or, Define multiple environments using variations of:
  - `docker-compose.yml` for development
  - `docker-compose.test.yaml` for testing
  - `docker-compose.prod.yaml` for production

By utilizing multiple configuration files for each environment, we are able to define custom settings (e.g. no web server needed during development), relevant variables (e.g. use test dataabse for testing), and services/containers (e.g. running a buidl/migration step for production).

Example: Wanting to setup both, a development and production configuration separately, we will create:

- `docker-compose.yml` for development
- `docker-compose.prod.yml` for production


## Version

The version keyword tells docker-compose what version of configurations we will be using. This helps with issues of using configurations, keywords, etc. that may have been used in earlier versions, but have since changed.

The first line of our docker-compose file should always be the version keyword.

Example: If we wanted to use version 3.7 (latest at the time of documentation), we would use:

```yaml
version: "3.7"
```

## Services

!!! warning "Services with ** require additional configuration variables defined on this page"
    Certain service parameters like Persistent Volumes and Shared Netowrks require additional configuration outside of the Services section. Services with this identifier will have their own section in this document.

The services section is where we define all the containers that we want Docker to create and their configurations/relations.

Each entry in the services section can have varying options and environment variables, which will be best defined on the core images' Dockerhub page.

Below is a matrix of popular parameters for services and wether they're required for base functionality:

| Parameter      | Required | Default Value | Description                                                                                                                                                                                                                 |
| -------------- | -------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| container_name | N        | random string | Custom name for container; extremely useful to identify containers that belong to a stack.                                                                                                                                  |
| image          | Y        |               | Image name to be used for container. This is replaced by `build` when using `Dockerfile` to build a custom image.                                                                                                           |
| build          | Y        |               | Path to [Dockerfile](./dockerfile.md). This is replaced by `image` when using pre-built imaged from Dockerhub                                                                                                               |
| restart        | N        | never         | Define if container should attempt to restart on fail or docker boot (e.g. system restart)                                                                                                                                  |
| environment    | N        |               | Environment variables that should be passed into the container.                                                                                                                                                             |
| ports          | N        |               | Port mapping from local system to container                                                                                                                                                                                 |
| volumes        | N        |               | Docker Volume and local filesystem mapping into the container. Mapping a local directory to the container (e.g. `./src`) will ensure the files are always in-sync, in both directions, even while the container is running. |
| networks       | N        |               | Shared network mapping between containers.**                                                                                                                                                                                |
| command        | N        |               | Custom command to be ran on container start/boot.**                                                                                                                                                                         |


If we wanted to define:

- Node service that builds from our Dockerfile
- MySQL database from a pre-built image in Dockerhub
- Allow containers to communicate between each other
- Ensure our DB data persists even when containers are shutdown/destroyed
- Expose our Node service to our local system on port 80

we would use:

```yaml
version: "3.7"

services:
  # our node application
  node_app:
    container_name: node_web            # custom name for the container
    build: .                            # use Dockerfile located project root
    restart: unless-stopped             # always restart, unless manually stopped
    environment:                        # define environment variables
      NODE_ENVIRONMENT: dev             
    ports:                              # map ports from local:container
      - 80:8000                         
    volumes:                            # map our source code into container workdir
      - ./src:/code                     
    networks:                           # use shared network between containers
      - node_db_network                 
  
  # mysql db for our node application
  node_db:
    container_name: node_db             # custom name for the container
    image: mysql:latest                 # name:version of pre-built image
    restart: unless-stopped             # always restart, unless manually stopped
    environment:                        # define environment variables
      MYSQL_ROOT_PASSWORD: root_pwd     
      MYSQL_DATABASE: app_db            
      MYSQL_USER: db_admin              
      MYSQL_PASSWORD: db_admin_pwd      
    volumes:                            # map persistent volume to DB store
      - node_db_volume:/var/lib/mysql   
    networks:                           # use shared network between containers
      - node_db_network                 
```


## Volumes

In docker-compose, volumes can be used to: 

- Create simple mappings of local code/config directories into the container,
- Map local directories to sync and retain data from within the container,
- Define Persistent Volumes that are named and kept even when containers are shutdown/recreated.

### Simple Volumes

In our examples we use a simple mapping to map our `./src` directory into the container `/code` work directory, which ensures our source code is always in-sync, allowing us to make live edits to code while the containers are running.

```yaml
volumes:
  - ./src:/code
```

### Persistent Volumes

For our database however, we do not have pre-existing data that we would want MySQL to use; instead, we would like to persist our data that is created once our application and database are up and running in Docker.

To achieve this, we will map out network similar to a Simple Volume, but add a separate configuration outside of the Services section:

```yaml
# our usual services definition, where we 
# map our persistent volume to the container
services:
  node_db:
    ...
    volumes:
      - node_db_network

# persistent volume definition
volumes:
  node_db_network:
```

## Networks

While we do define and expose ports on the individual containers to our local system, there are cases where we want certain containers to communicate with each other, and only each other.

To achieve this, we create named networks and then define which containers have access to them.

If we wanted to let our node_web container communicate with the database in node_db container, without exposing it to our local system, we would use:

```yaml
# our usual services definition, where we map our 
# named network to the containers that are allowed to use it
services:
  node_web:
    ...
    networks:
      - node_db_network 
  node_db:
    ...
    networks:
      - node_db_network 

# named network definition
networks:
  node_db_network:
```


## Completed Example

```yaml
version: "3.7"

services:
  # our node application
  node_app:
    container_name: node_web            # custom name for the container
    build: .                            # use Dockerfile located project root
    restart: unless-stopped             # always restart, unless manually stopped
    environment:                        # define environment variables
      NODE_ENVIRONMENT: dev             
    ports:                              # map ports from local:container
      - 80:8000                         
    volumes:                            # map our source code into container workdir
      - ./src:/code                     
    networks:                           # use shared network between containers
      - node_db_network                 
  
  # mysql db for our node application
  node_db:
    container_name: node_db             # custom name for the container
    image: mysql:latest                 # name:version of pre-built image
    restart: unless-stopped             # always restart, unless manually stopped
    environment:                        # define environment variables
      MYSQL_ROOT_PASSWORD: root_pwd     
      MYSQL_DATABASE: app_db            
      MYSQL_USER: db_admin              
      MYSQL_PASSWORD: db_admin_pwd      
    volumes:                            # map persistent volume to DB store
      - node_db_volume:/var/lib/mysql   
    networks:                           # use shared network between containers
      - node_db_network

# persistent volume definition
volumes:
  node_db_network:

# named network definition
networks:
  node_db_network:
```