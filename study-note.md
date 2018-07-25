## Docker Container commands

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