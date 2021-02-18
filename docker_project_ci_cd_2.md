# Continuous Integration and Deployment (CI/CD 2)

```
GitHub [feature branch ---> master] ---> CI ---> AWS (elastic beanstalk)
```

Changes in feature branch merged into master triggers CI to launch docker to run test and build docker image for production and deploy to AWS (docker platform) automatically

GitHub

https://github.com/jerryhwg/docker-react

Workflow

1. Tell Travis we need a copy of docker running (.travis.yml)
    
    ```
    sudo 
    services: docker
    ```

2. Build our image using Dockerfile.dev

    ```
    docker build -f Dockerfile.dev
    ```

3. Tell Travis how to run our test suite (npm run test)

    ```
    docker run -e CI=true -t docker-react npm run test -- --coverage
    ```

4. Tell Travis how to deploy our code to AWS

    ```
    Elastic Beanstalk

    Platform:
    - Docker
    - Docker running on 64bit Amazon Linux
    - 2.16.4 (Recommended)
    - Sample application
    ```