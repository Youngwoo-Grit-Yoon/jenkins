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
5. 하기 명령어를 이용하여 myjenkins-blueocean:2.375.2-1 이미지의 컨테이너를 실행합니다.
```text
docker run \
  --name jenkins-blueocean \
  --restart=on-failure \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.375.2-1 
```
**Note:** 주석이 없는 하기 명령어를 이용하여 실행할 수 있습니다.
```text
docker run --name jenkins-blueocean --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.375.2-1
```
6. Post-installation setup wizard(<https://www.jenkins.io/doc/book/installing/docker/#setup-wizard>)를 진행합니다.
## Post-installation setup wizard
위자드를 통해서 Jenkins를 언락하고 플러그인을 통해 커스터마이징을 수행하며 Jenkins에 접근할 수 있는 첫 번째 Admin 계정을 생성할 수
있습니다.
### Unlocking Jenkins
맨 처음 Jenkins 인스턴스에 접근하면 자동으로 생성된 패스워드를 이용하여 언락하도록 요청을 받습니다.
1. `http://localhost:8080` 주소로 접근하여 **Unlock Jenkins** 페이지가 나타날 때까지 대기합니다.
2. Jenkins 콘솔 로그에서 자동으로 생성된 alphanumeric 패스워드를 복사합니다.
   - 만약 공식 jenkins/jenkins Docker 이미지를 이용하여 Jenkins를 실행 중이라면 하기 명령어를 이용하여 패스워드를 확인할 수 있습니다.  
   `sudo docker exec ${CONTAINER_ID or CONTAINER_NAME} cat /var/jenkins_home/secrets/initialAdminPassword`
3. 복사한 패스워드(dcd742ec21a34cb68851ca0921a8c61e)를 기입하여 **Continue** 버튼을 클릭합니다.