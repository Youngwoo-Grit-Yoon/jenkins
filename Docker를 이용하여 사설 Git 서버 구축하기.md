## Docker를 이용하여 사설 Git 서버 구축하기
Base 이미지 CentOS:7.9.2009를 이용하여 Git을 설치할 컨테이너를 생성한다.
```shell
docker run --name git-server \
-d \
--privileged \
-p 6000:22 \
centos:7.9.2009 \
init
```
하기 명령어를 수행하여 git-server 컨테이너 내부로 이동한 후 Git을 설치한다.  
```text
[root@localhost ~]# docker exec -it git-server /bin/bash
[root@3a0707c64760 /]# yum install -y git
... 생략 ...
Complete!
```
