## Docker를 이용하여 사설 Git 서버 구축하기
### git-server 컨테이너 생성 및 SSH 서버 설치
Base 이미지 CentOS:7.9.2009를 이용하여 Git을 설치할 컨테이너를 생성한다.  
6000번 포트를 컨테이너 내부의 SSH 서비스 22번 포트에 바인딩 한다.
```shell
docker run --name git-server \
-d \
--privileged=true \
-p 6000:22 \
centos:7.9.2009 \
init
```
git-server 컨테이너 내부로 이동한다.
```shell
docker exec -it git-server /bin/bash
```
패키지를 최신화 한다.
```shell
yum install -y epel-release
```
컨테이너 내부 root 계정의 비밀번호를 설정한다.
```shell
echo 'root:password' | chpasswd
```
SSH를 설치한다.
```shell
yum install -y openssh-server
```
sshd 설정 파일을 수정한다.
```text
vi /etc/ssh/sshd_config
... 생략 ...
#Port 22 --> 주석 제거
```
SSH 서비스 재기동 후 SSH 접근을 시도해본다.
```shell
systemctl restart sshd
```
```shell
ssh -l root -p 6000 localhost
```
### Git 설치
Git을 설치한다.  
```shell
yum install -y git
```
```text
[root@d880e49ceaaa /]# git --version
git version 1.8.3.1
```
저장소를 생성한다.
```text
[root@d880e49ceaaa opt]# mkdir -p /opt/git/youngwoo/youngwoo.git
[root@d880e49ceaaa opt]# git init --bare /opt/git/youngwoo/youngwoo.git/
Initialized empty Git repository in /opt/git/youngwoo/youngwoo.git/
```
계정을 생성한다.
```text
useradd youngwoo -m -s /bin/bash
echo 'youngwoo:password' | chpasswd
chown -R youngwoo:youngwoo /opt/git/youngwoo
```
저장소 복사를 시도해본다.
```text

```