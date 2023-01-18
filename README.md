# Jenkins
## What is Jenkins
Jenkins는 소프트웨어의 빌드, 테스트, 배포에 관련된 모든 종류의 업무를 자동화하는 데 사용되는 오픈 소스 자동화 서버입니다.
## Installing Jenkins using Docker
1. 터미널 윈도우를 실행합니다.
2. 다음 명령어를 실행하여 Docker에 브릿지 네트워크를 생성합니다.
```text
docker network create jenkins
```
3. Jenkins 노드 내부에서 Docker 명령어를 실행하기 위해서는 다음 명령어를 실행하여 `docker:dind` 이미지를 다운로드 해야 합니다.
```text
docker run \
  --name jenkins-docker \ (1)
  --rm \ (2)
  --detach \ (3)
  --privileged \ (4)
  --network jenkins \ (5)
  --network-alias docker \ (6)
  --env DOCKER_TLS_CERTDIR=/certs \ (7)
  --volume jenkins-docker-certs:/certs/client \ (8)
  --volume jenkins-data:/var/jenkins_home \ (9)
  --publish 2376:2376 \ (10)
  docker:dind \ (11)
  --storage-driver overlay2 (12)
```
(1) Docker 컨테이너 이름을 지정합니다.  
(2) 컨테이너가 종료되었을 때 자동으로 해당 컨테이너를 제거합니다.  
(3) Docker 컨테이너를 백그라운드로 실행합니다.