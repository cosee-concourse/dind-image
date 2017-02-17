# Docker-in-Docker Image [![Docker Repository on Quay](https://quay.io/repository/cosee-concourse/dind/status "Docker Repository on Quay")](https://quay.io/repository/cosee-concourse/dind)

This image lets you run Docker within Docker.

Additionally this image supports Docker Compose.

This image is hosted at quay.io: [quay.io/cosee-concourse/dind](https://quay.io/cosee-concourse/dind)
## Example: Usage with ConcourseCI
### Docker Compose
Pipeline definition:
```yaml
...

jobs:
- name: docker-compose-job
  plan:
  - get: source
    trigger: true
  - task: runDockerCompose
    privileged: true        # required
    file: source/path_to_job_definition/jobDefinition.yml
    
... 
```

Job definition (e.g. jobDefinition.yml):

``` yaml
platform: linux

image_resource:
        type: docker-image
        source: {
        repository: quay.io/cosee-concourse/dind,
        tag: "latest",
        privileged: true }

run:
        path: sh
        args:
        - -exc
        - |
          source /docker-lib.sh               # required
          start_docker                        # required
          cd source/path_to_dockercompose_yml
          docker-compose up -d
          # execute your tasks e.g.
          docker-compose -f docker-compose-runTasks.yml run testservice echo "Hello World"
          rc=$?                               # exit code of testservice
          docker-compose down                 # required
          exit $rc

inputs:
    - name: source
```

### Docker in Docker
Pipeline definition:
```yaml
...

jobs:
- name: docker-job
  plan:
  - get: source
    trigger: true
  - task: runDockerContainer
    privileged: true        # required
    file: source/path_to_job_definition/jobDefinition.yml
    
... 
```

Job definition (e.g. jobDefinition.yml):

``` yaml
platform: linux

image_resource:
        type: docker-image
        source: {
        repository: quay.io/cosee-concourse/dind,
        tag: "latest",
        privileged: true }

run:
        path: sh
        args:
        - -exc
        - |
          source /docker-lib.sh               # required
          start_docker                        # required
          # for own Dockerfiles:
          cd source/path_to_dockerfile
          docker build -t testimage .
          docker run -it --rm testimage parameter
          # other example: use image from Docker Hub:
          docker run -it --rm ubuntu echo "Hello World"

inputs:
    - name: source
```
