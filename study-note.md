## Docker Container

```bash
# use nginx image as an example
docker container run --publish 80:80 nginx

# `--publish`: publish (map) container port to the host `<container>:<host>`
# `--name`: give container a name (basically slug, id)
# `--detach`: run container in the background
# `--env FOO=bar`: provide environment variable
docker container run --publish 80:80 --name webserver --detach nginx

# automatically remove container when it exits
docker container run --rm ...
# good for running a one-off command like:
docker container run --rm --network my_net alpine nslookup search

docker container ls
docker container stop <name | container_id>
docker container logs <name | container_id>
docker container top <name | container_id>
# check configuration for starting container
docker container inspect <name | container_id>
# check how the container running (good for local machine during dev)
docker container stats

# run container with interactive tty and start with bash
docker container run -it nginx bash

# attach a new process to a running container
# with interactive tty (e.g. use bash program)
docker container exec -it nginx bash
# can also pass argument to the command like:
docker container exec -it nginx ping www.google.com

# start existing container with interactive tty
# does not accept command - will use command from "run"
docker container start -ia nginx
```

Alpine Linux is a small, security-focused distro - commonly used.
Alpine does not contain bash, use `sh` instead.
`apk` is the package manager that comes with alpine.

```bash
docker pull alpine
```

## Docker Network

- a container can be on multiple networks
- a container on the same network can see and refer to each other by name (container name)

```bash
docker network ls
docker network inspect <network_name>

# create a new custom network
# `--driver` option can be passed
docker network create <network_name>

# connect a running container to a network
# note: a container can be in multiple networks
docker network connect <network_name> <container_name>
docker network disconnect <network_name> <container_name>

# create container using custom network
docker container run  ... --network <network_name> ...

docker container run ... --network-alias
```

## Docker Image

- an image is an app binary and dependencies together with metadata how to run it
- inside an image, no os or kernel (host provides it) just binaries

```bash
docker image ls
docker image rm <name>
docker image history <name>
docker image inspect <name>

# create a new tag based on an image
# can also be used to "rename" local tag to be suitable to push to image registry
docker image tag <source_image[:tag]> <target_image[:tag]>

docker image push <image_name[:tag]>
docker image pull <image_name[:tag]>

# create a new image from a Dockerfile
# `-t` tag image with image_name[:tag]
# `.` path: uses the Dockerfile in this directory
docker image build -t <image_name[:tag]> .
```

## Dockerfile

- remember: Dockerfile is for creating "images"
- use `-f` to use a different docker filename than the default `Dockerfile`
- each command creates a new layer
- keep layers that change least often at the top and layers that change most often at the bottom
- `FROM` is required. Denote source image to extend. Target image will not inherit `ENV` from the source image.
- `ENV` key-value environment variables
- `RUN` should use commands chaining `&&` if you don't want to create a separate layer
- `WORKDIR` change directory like doing `cd` but it's a good practice to use this rather than `RUN cd ...`
- `COPY` just copy file from the cwd in host to the cwd in the container
- `CMD`, final command that will be run anytime the container is started
- Dockerfile builder reference: https://docs.docker.com/engine/reference/builder/
- best practice: remove package manager cache after installing dependencies
  - `apt-get update && apt-get install -y <package> && rm -rf /var/lib/apt/lists/*`
  - `yarn install --pure-lockfile && yarn run prune && yarn cache clean`
- best practice: git clone only what you need
  - `git clone --branch <my_branch> --single-branch --depth 1 ...`
- misc: change permission `RUN chown -R <user>:<group> in_directory`

## Persistent Data

- you can mount code into the container from host while editing it so you can run the code in the container, live

### Volume

- creates volume outside container union file-system
- is persisted after container removal (needs manual deletion)
- is not shared across containers
- inspect with `docker container inspect <container_name>`
- in Dockerfile: `VOLUME /path/to/volume` (creates or use existing)

```bash
docker volume prune # clean up
# starts the container with named volume
docker container run ... -v <volume_name:/path/to/volume_in_container> ...
docker volume create
```

### Bind Mount

- link host directorys/files to the container
- is not part of container union file-system (won't change layer)
- cannot be used in Dockerfile, must be at `docker container run`
- creating, editing, removing file inside the container _will_ affect the host

```bash
# bind mount, can use `$(pwd)` <- standard shell stuff like:
# `-v $(pwd)/file.ext:/path/container/file.ext`
docker container run ... -v </path/on/host:/path/container>
```

## Docker Compose

- docker-compose cli is not meant for production
- easy to set up dev environment (tooling), the project only needs `Dockerfile` and `docker-compose.yml` then run `docker-compose up`
- a default network will be created and all containers in the compose file will share it by default
- `docker-compose down` is essential for cleanup

```bash
# use -f to specify the compose file to use
docker-compose up # set up volumes, networks and start all containers

docker-compose up --build
docker-compose build

# *DANGER* use -v to remove volumes as well
docker-compose down  # stop all containers and removed as well as volumes, networks
```

```yaml
# https://docs.docker.com/compose/compose-file/compose-versioning/#compatibility-matrix
version: '3.1'  # if no version is specificed then v1 is assumed. Recommend v2 minimum

# a service can have multiple containers
services:  # containers. same as docker container run
  servicename: # a friendly name. this is also DNS name inside network
    image: # Optional if you use build:
    command: # Optional, replace the default CMD specified by the image
    environment: # Optional, same as -e in docker container run
    # `.` for volumes refer to cwd - e.g. `.:/usr/shared/data`
    # appends `:ro` means read-only - e.g. `./package.json:/home/source/package.json:ro`
    volumes: # Optional, same as -v in docker container run
    ports: # expose ports
    depends_on: # service names (in this file) that this service depends on
    # Optional, build custom image - need to manually rebuild if it's existed
    build:
      context: .
      dockerfile: service.Dockerfile
  servicename2:

volumes: # Optional, same as docker volume create

networks: # Optional, same as docker network create
```

## Misc Notes

- one machine (or VM) one docker daemon (share ports) multiple containers