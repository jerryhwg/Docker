# Docker Project Example : NodeJS

NodeJS

1. Createa NodeJS web app
2. Create a Dockerfile
3. Build image from dockerfile
4. Run image as container
5. Connect to web app froma a browser

Dockerfile

```docker
# Specify a base image
FROM node:alpine

WORKDIR /usr/src/app

# Install some dependenices
COPY ./package.json ./
RUN npm install
RUN npm install express
COPY ./ ./

# Default command
CMD ["npm", "start"]
```

Build

```console
docker build -t jerryhwang72/simpleweb .
```

Run

```console
docker run -p 9001:8080 jerryhwang72/simpleweb
```

* 9001: route incoming requests to this port on local host to
* 8080: port inside the container