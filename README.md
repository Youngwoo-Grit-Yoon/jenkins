# Jenkins
## What is Jenkins
Jenkins는 소프트웨어의 빌드, 테스트, 배포에 관련된 모든 종류의 업무를 자동화하는 데 사용되는 오픈 소스 자동화 서버이다.
## Installing Jenkins using Docker
1. 터미널 윈도우를 실행한다.
2. 다음 명령어를 실행하여 Docker에 브릿지 네트워크를 생성한다.
```shell
docker network create jenkins
```
3. Jenkins 노드 내부에서 Docker 명령어를 실행하기 위해서는 다음 명령어를 실행하여 `docker:dind` 이미지를 다운로드 해야 합니다.
```shell
docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2
```
