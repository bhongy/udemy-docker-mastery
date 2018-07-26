## Docker Container

```bash
# use nginx image as an example
docker container run --publish 80:80 nginx

# `--publish`: publish (map) container port to the host
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

## Misc Notes

- one machine (or VM) one docker daemon (share ports) multiple containers