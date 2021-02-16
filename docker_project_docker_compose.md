# Docker Project Example: Docker Compose & Multiple Local Containers

Docker Container ( NodeJS App ) ---> Docker Container ( Redis )

Dockerfile

```docker
FROM node:alpine

WORKDIR '/app'

COPY package.json .

RUN nmp install

COPY . .

CMD ["npm", "start"]
```

Build

```console
docker build -t jerryhwang72/visits .
```

Run

```console
docker run jerryhwang72/visits
```

### Use docker compose

```docker
version: '3'
services:
    redis-server:
        image: 'redis'
    node-app:
        build: .
        ports:
            - "8081:8081"
```

docker-compose CLI

```console
docker build -t jerryhwang72/visits:latest
docker run -p 8081:8081 jerryhwang72/visits
--->
docker-compose.yml
--->
docker-compose up [--build]
```
