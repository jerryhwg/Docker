# Multi-Container App on Docker

## Build a Multi-Container App

Challenges

* Outside dependenices
* Image is built multiple times
* Connect to a database


Build App

```
Nginx -> React Server(front-end) / Express Server(backend API)  -> Redis(Cache)[<-> Worker] / Postgres(permanent DB)
```

### Worker

**NodeJS**

watches redis for new indices, pulls each new indice, calculates new value then puts it back into redis

package.json

```javascript
{
    "dependenices": {
        "nodemon": "1.18.3"
        "redis": "2.8.0" # redis client
    },
    "scripts": {
        "start": "node index.js",
        "dev": "nodemon"
    }
}
```

index.js

```javascript
const keys = require('./keys');
const redis = require('redis');

const redisClient = redis.createClient({
    host: keys.redisHost,
    port: keys.redisPort,
    retry_strategy: () => 1000
});
const sub = redisClient.duplicate();

function fib(index)

sub.on('message', (channel, message) => {
    redisClient.hset('values', message, fib(parseInt(message)));
});
sub.subscribe('insert');
```

keys.js

```javascript
module.exports = {
    redisHost: process.env.REDIS_HOST,
    redisPort: process.env.REDIS_PORT
};
```

### Redis

stores all indices and calculated values as key-value pairs

### Postgres

stores a permanent list of indices that have been received

### Express Server

Server (express)

Backend API to redis and postgres

package.json

```javascript
{
    "dependenices": {
        "express": "4.16.3",
        "pg": "7.4.3",
        "redis": "2.8.0",
        "cors": "2.8.4",
        "nodemon": "1.18.3"
    },
    "scripts": {
        "dev": "nodemon",
        "start": "node index.js"
    }
}
```

keys.js

```javascript
module.exports = {
    redisHost: process.env.REDIS_HOST,
    redisPort: process.env.REDIS_PORT,
    pgUser: process.env.PGUSER,
    pgHost: process.env.PGHOST,
    pgDatabase: process.env.PGDATABASE,
    pgPassword: process.env.PGPASSWORD,
    pgPort: process.env.PGPORT
};
```

index.js

```javascript
// Express App Setup

// Postgres Client Setup
const { Pool } = require('pg');
const pgClient = new Pool({
    user: keys.pgUser,
    host: keys.pgHost,
    database: keys.pgDatabase,
    password: keys.pgPassword,
    port: keys.pgPort
});

pgClient.query('CREATE TABLE IF NOT EXISTS values (number INT)')

// Redis Client Setup
const redis = require('redis');
const redisClient = redis.createClient({
    host: keys.redisHost,
    port: keys.redisPort,
    retryStrategy: () => 1000
});

// Express route handlers
app.get('/', (req, res) => {
    res.send('Hi');
});

app.get('/values/all', async (req, res) => {
    const values = await pgClient.query('SELECT * from values')

    res.send(values.rows);
});

app.get('/values/current', async (req, res) => {
    redisClient.hgetall('values', (err,values) => {
        res.send(values);
    });
});

...
...

app.listen(5000, err => {
    console.log('Listening');
});
```

### React App

Client (React)

Front-end (html, css, js)

```
npx create-react-app client
cd client
rm -r .git
```

```javascript
import React, { Component } from 'react';
import axios from 'axios';
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

