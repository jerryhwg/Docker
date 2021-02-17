# Continuous Integration and Deployment

```
GitHub [feature branch ---> master] ---> CI ---> AWS
```

GitHub

https://github.com/jerryhwg/docker-react

Workflow

1. Tell Travis we need a copy of docker running
    
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