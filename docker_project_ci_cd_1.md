# Docker Project Example: React (CI/CD 1)

Install NodeJS

https://nodejs.org/en/download/

```
npm run start : start up a development server
npm run test : run tests associated with the project
npm run build : build a production version of the application
```

Create React App generation

```console
cd ~/project
npx create-react-app frontend
```

`NOTE: you should see 'Happy hacking!' at the end.`

```console
cd frontend
npm run test
# select a
npm run build
ls build/static/js
# see main.xxx.js
npm run start
# a browser will start
```

Create a Dev Dockerfile

```docker
FROM node:alpine

WORKDIR '/app'

COPY package.json

RUN nmp install

COPY . .

CMD ["npm", "run", "start"]
```

build

```console
docker build -f Dockerfile.dev .
```

`NOTE: remove 'node_modules' after the intial build to speed up`

run

```console
docker run -it -p 3000:3000 <image id>
```

access: open a browser at http://localhost:3000

Docker volume

```console
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <image id>
```

`NOTE: -v /app/node_modules : put a bookmark on the node_modules folder`

Utilize docker-compose

docker-compose.yml

```docker
version: '3'
services:
    web:
        build:
            context: .
            dockerfile: Dockerfile.dev
        ports:
            - "3000:3000"
        volumes:
            - /app/node_modules
            - .:/app
```

run

```console
docker-compose up
```

run test

```console
docker run -it -p 3000:3000 <image id> nmp run test
```

docker-compose.yml

```docker
version: '3'
services:
    web:
        build:
            context: .
            dockerfile: Dockerfile.dev
        ports:
            - "3000:3000"
        volumes:
            - /app/node_modules
            - .:/app
    tests:
        build:
            context: .
            dockerfile: Dockerfile.dev
        volumes:
            - /app/node_modules
            - .:/app
        command: ["npm", "run", "test"]
```

Build a production version of a container using nginx

Build phase

1. use node: alpine
2. copy package.json file
3. install dependencies
4. run "npm run build"

Run Phase

1. Use nginx image
2. Copy over the result of 'npm run build'
3. Start nginx

Dockerfile

```docker
FROM node:alpine
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build
 
FROM nginx
COPY --from=0 /app/build /usr/share/nginx/html
```

run

```console
docker run -it -p 8080:80 <image id>
```