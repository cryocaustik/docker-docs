# Dockerfile

Dockerfile is the core configuration file used to define custom built images/containers.

If your stack is using pre-built images only (e.g. MySQL, node, etc.), then you would not need this file.

However, if you're looking to run a Node server with your custom code, we would need to build a new image, mixing our code onto an existing Node image, and running that in a container.


## Keywords

Below we will talk about the sections of a Dockerfile and how to use them.

### FROM

!!! warning "Use Trusted Sources Only"
    Core images should only be used from official sources such as [Dockerhub](https://hub.docker.com/), as images will contain base code that will run in your container.

The first line of any Dockerfile should always define the core image we're starting from.

Using an all caps `FROM` keyword, we can then specify the image, version, and variant, separating the image name and version/type with a semi-colo (`:`).

Available image versions and variants should always be defined on the Image page of Dockerhub.

Example: For a Node application, we would use a **[node:lts](https://hub.docker.com/_/node)** command to pull the latest LTS version of the Node image.

```dockerfile
FROM "node:lts"
```

### RUN

The run command is used to define shell commands that should be executed at the time of compiling/building our custom image.

These commands are typically used to:

- Adding missing/custom repositories for APT
- Install missing dependencies (e.g. missing mysql driver)
- Creating custom directories

Example: if we wanted to create a custom directory for our application source code and run an APT update/upgrade to ensure the core image is up to date, we would use:

```dockerfile
RUN mkdir /code
RUN apt update && apt upgrade -y
```


### WORKDIR

The workdir keyword allows us to specify where our source code will live and most importantly, where to run our commands from, within the image.

It is important to ensure our workdir directory exists before we specify this keyword, as the process will attempt to find and link to it.

Example: If I wanted to define my wordir as `/code`, I would use:

```dockerfile
RUN mkdir /code
WORKDIR /code
```

### COPY

The copy command is used to copy files/directories into the image.

Copying files into the image at compile, can be very helpful in setting up environments and installing pre-requisites that do not need to be ran every time a container is started.

Example: If we wanted to copy our package.json config into the image and install the required Node packages, we would use:

```dockerfile
COPY package.json /code/
RUN yarn install
```

### EXPOSE

!!! info "Expose Not used when using Docker Compose"
    While the expose command can be used to expose ports in the container, it is typically not used when a stack is defined using docker-compose.yml, as it compose will have it's own mapping that is better defined.

The expose command can be used to expose container ports to the local environments.

Example: if we wanted to expose port 8000 to our local machine, we would use:

```dockerfile
EXPOSE 8000
```

## Completed Example

```dockerfile
# define core image:version/variant
FROM "node:lts-alpine"

# make our source code directory
RUN mkdir /code

# define our source code directory as the root for commands
WORKDIR /code

# copy our package.json config into the image
COPY package.json /code/

# install our node package dependencies
RUN yarn install

# perform an initial copy of our source code into the image
COPY ./src /code/

# production use: build our node source code
RUN yarn build
```