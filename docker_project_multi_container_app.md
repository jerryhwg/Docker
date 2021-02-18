# Multi-Container App on Docker

## Build a Multi-Container App

Challenges

* Outside dependenices
* Image is built multiple times
* Connect to a database


Build App

```
Nginx -> React Server(front-end) / Express Server(backend API)  -> Redis(Worker) / Postgres(permanent DB)
```

### Worker

NodeJS

watches redis for new indices, pulls each new indice, calculates new value then puts it back into redis

### Redis

stores all indices and calculated values as key-value pairs

### Postgres

stores a permanent list of indices that have been received

### Express Server

Server (express)

backend API to redis and postgres

### React App

Client (React)

front-end (html, css, js)

```
npx create-react-app client
cd client
rm -r .git
```

## Dockerizing Apps

React App, Express Server, Worker

1. copy over package.json
2. run nmp install
3. copy over everything else (src, index.js)
4. docker-compose to set up a volume to 'share' files

### React App (Client)

```docker
FROM node:alpine
WORKDIR '/app'
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "start"]
```

```console
cd client
docker build -f Dockerfile.dev .
docker run -it -p 3000:3000 <image id>
```

### Node Apps

```docker
FROM node:alpine
WORKDIR '/app'
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]
```

### Nginx Path Routing

default.conf

```
* Tell Nginx that there is an 'upstream' server at client:3000
* Tell Nginx that there is an 'upstream' server at server:5000

===>

* Listen on port 80
* If anyone comes to '/' send them to client upstream
* If anyone comes to '/api' send them to server upstream
```

Dockerfile

```docker
FROM nginx
COPY default.conf /etc/nginx/conf.d/default.conf
```

### docker-compose

docker-compose.yml

```docker
version: '3'
services:
    postgres:
        image: 'postgres:latest'
        environment:
            - POSTGRES_PASSWORD=
    redis:
        image: 'redis:latest'
    nginx:
        depends_on:
            - api
            - client
        restart: always
        build:
            dockerfile: Dockerfile.dev
            context: ./nginx
        ports:
            - '3050:80'
    api:
        build:
            dockerfile: Dockerfile.dev
            context: ./server
        volumes:
            - /app/node_modules
            - ./server:/app
        environment:
            - REDIS_HOST=redis
            - REDIS_PORT=6379
            - PGUSER=postgres
            - PGHOST=postgres
            - PGDATABASE=postgres
            - PGPASSWORD=
            - PGPORT=5432
    client:
        stdin_open: true
        build:
            dockerfile: Dockerfile.dev
            context: ./client
        volumes:
            - /app/node_modules
            - ./client:/app
    worker:
        build:
            dockerfile: Dockerfile.dev
            context: ./worker
        volumes:
            - /app/node_modules
            - ./worker:/app
        environment:
            - REDIS_HOST=redis
            - REDIS_PORT=6379 
```

run

```console
docker-compose up [--build]
```

