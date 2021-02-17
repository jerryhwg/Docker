# Docker How To

Docker Desktop

* [Docker for windows 10 Professional](https://hub.docker.com/editions/community/docker-ce-desktop-windows/)
* [Docker for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac/)

Image: single file (FS snapshot) with all the deps and config required to run a program

Container: instance of an image, run a program

## Docker Client

docker run

```console
docker run <image name> [command]
```

`docker run = docker create <image name> + docker start <container id>`

docker start and attached

```console
docker start -a <container id>
```

```console
docker start <container id>
docker logs <container id>
```

stop container

```console
# SIGTERM (10 seconds, recommended)
docker stop <container id>
# SIGKILL (immediate, fallback for stop)
docker kill <container id>
```

list all running containers

```console
docker ps
docker ps -a
```

removing (cleaning up) stopped containers

```console
docker system prune
docker ps -a
```

execute an additional command in a container

```console
docker exec -it <container id> [command]
```

`-it: allow us to provide input to the container`

> example

```console
docker run redis
docker exec -it <container id> redis-cli
docker exec -it <container id> sh
```

starting with a shell

```console
docker run -it busybox sh
```

## Docker Server

Creating docker image

> Dockerfile -> Docker Client -> Docker Server -> Docker Image

Dockerfile: configuration to define how our ctoaniner should behave

Creating a Dockerfile

1. Specify a base image
2. Run some commands to install additional programs
3. Specify a command to run on container startup

> Example: Dockerfile

```docker
# Use an existing docker image as a base
FROM alpine

# Download and install a dependency
RUN apk add --update redis

# Tell the image what to do when it starts as a container
CMD ["redis-server"]
```

```console
docker build .
# get image id
docker run <image id>
```

Rebuilds with cache

Tagging an image

```console
docker build -t jerryhwang72/redis:latest .
docker run jerryhwang72/redis
```

* jerryhwang72: docker id
* redis: repo/project name
* latest: version

`NOTE: docker build -t jerryhwang/app-dev -f Dockerfile.dev .`

(Optional) Create a docker image with docker commit

```console
docker run -it alpine sh
# inside shell
$ apk add --update redis
# open a second terminal
# Mac
docker commit -c 'CMD ["redis-server"]' <container id>
# Windows
docker commit -c "CMD 'redis-server'" <container id>
```