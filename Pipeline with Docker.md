## Pipeline with Docker
### Customizing the execution environment
**Note**: 하기 컨테이너 두 개가 실행 중인 상태인지 확인한다.
```text
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS          PORTS                                                                                      NAMES
71802f561a6a   myjenkins-blueocean:2.375.2-1   "/usr/bin/tini -- /u…"   19 minutes ago   Up 19 minutes   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:50000->50000/tcp, :::50000->50000/tcp   jenkins-blueocean
35b497b59053   docker:dind                     "dockerd-entrypoint.…"   19 minutes ago   Up 19 minutes   2375/tcp, 0.0.0.0:2376->2376/tcp, :::2376->2376/tcp                                        jenkins-docker
```
```text
pipeline {
    agent {
        docker { image 'node:16.13.1-alpine' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
            }
        }
    }
}
```
### Workspace synchronization
https://www.jenkins.io/doc/book/pipeline/docker/#workspace-synchronization
### Caching data for containers
https://www.jenkins.io/doc/book/pipeline/docker/#caching-data-for-containers
### Using multiple containers
https://www.jenkins.io/doc/book/pipeline/docker/#using-multiple-containers
### Using a Dockerfile
For projects which require a more customized execution environment, Pipeline also supports building and running a
container from a Dockerfile in the source repository. In contrast to the previous approach of using an "off-the-shelf"
container, using the agent { dockerfile true } syntax will build a new image from a Dockerfile rather than pulling one
from Docker Hub.  
Re-using an example from above, with a more custom `Dockerfile`:
```text
Dockerfile
FROM node:16.13.1-alpine

RUN apk add -U subversion
```
By committing this to the root of the source repository, the `Jenkinsfile` can be changed to build a container based on
this `Dockerfile` and then run the defined steps using that container:
```text
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent { dockerfile true }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
                sh 'svn --version'
            }
        }
    }
}
```
The agent { dockerfile true } syntax supports a number of other options which are described in more detail in the
Pipeline Syntax section.
### Building containers
https://www.jenkins.io/doc/book/pipeline/docker/#building-containers
### Using a remote Docker server
https://www.jenkins.io/doc/book/pipeline/docker/#using-a-remote-docker-server
### Using a custom registry
https://www.jenkins.io/doc/book/pipeline/docker/#custom-registry