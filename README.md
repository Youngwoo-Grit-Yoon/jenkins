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
(4) 권한을 부여합니다.  
(5) 앞 단계에서 생성한 브릿지 네트워크를 사용합니다.  
(6) Docker 컨테이너 내부의 Docker를 jenkins 네트워크에서 호스트 이름 docker로 사용 가능하도록 합니다.  
(7) Docker 서버에서 TLS의 사용을 활성화 합니다.  
(8) 컨테이너 내부의 /certs/client 디렉토리를 상기에서 생성한 jenkins-docker-certs Docker 볼륨에 매핑합니다.  
(9) 컨테이너 내부의 /var/jenkins_home 디렉토리를 상기에서 생성한 jenkins-data Docker 볼륨에 매핑합니다.  
(10) Docker 데몬 포트를 호스트 머신에 노출시킵니다.  
(11) docker:dind 이미지  
(12) Docker 볼륨을 위한 스토리지 드라이버  
**Note:** 주석이 없는 하기 명령어를 이용하여 실행할 수 있습니다.
```text
docker run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind --storage-driver overlay2
```
4. 하기 두 단계를 통해서 공식 Jenkins Docker 이미지의 커스텀을 수행합니다.
   1. 하기 내용을 이용하여 Dockerfile을 생성합니다.
   ```text
   FROM jenkins/jenkins:2.375.2
   USER root
   RUN apt-get update && apt-get install -y lsb-release
   RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
    https://download.docker.com/linux/debian/gpg
   RUN echo "deb [arch=$(dpkg --print-architecture) \
    signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
    https://download.docker.com/linux/debian \
    $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
   RUN apt-get update && apt-get install -y docker-ce-cli
   USER jenkins
   RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
   ```
   2. 상기 Dockerfile을 이용하여 새로운 Docker 이미지를 빌드하고 의미있는 이름을 부여합니다.
   ```text
   docker build -t myjenkins-blueocean:2.375.2-1 .
   ```
   참고로 위 단계를 최초로 수행할 때 공식 Jenkins Docker 이미지를 다운로드할 것입니다.
