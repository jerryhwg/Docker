# Multi-Containers CI/CD

## Workflow

1. Push code to GitHub (Dockerfile)
2. Travis automatically pulls repo
3. Travis builds a test image, test code
4. Travis builds prod images
5. Travis pushes built prod images to Docker Hub
6. Travis pushes project to AWS EB
7. EB pulls images from Docker Hub, deploys

Instead of using AWS EB to build image, pull prod images from Docker Hub.

### Travis CI

.travis.yml

1. Specify docker as dependency
2. Build test version of React project
3. Run tests
4. Build prod versions of all projects
5. Push all to docker hub
6. Tell EB to update

```yaml
sudo: required
services:
  - dockerfile

before_install:
  - docker build -t jerryhwang72/react-test -f ./client/Dockerfile.dev ./client

script:
  - docker run -e CI=true jerryhwang72/react-test npm run test -- --coverage

after_success:
  - docker build -t jerryhwang72/multi-client ./client
  - docker build -t jerryhwang72/multi-nginx ./nginx
  - docker build -t jerryhwang72/multi-server ./server
  - docker build -t jerryhwang72/multi-worker ./worker

  # Log in to the docker CLI
  - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_ID" --password-stdin
  # Take those images and push them to docker hub
  - docker push jerryhwang72/multi-client
  - docker push jerryhwang72/multi-nginx
  - docker push jerryhwang72/multi-server
  - docker push jerryhwang72/multi-worker
```

### Deploy to AWS EB