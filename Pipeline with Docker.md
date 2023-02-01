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
