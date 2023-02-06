## Jenkins-Git 연동하는 법
(1) item을 하나 생성한다.  
![](./img/scm1.png)  

(2) 하기와 같이 SCM에서 Git을 선택한다.  
![](./img/scm2.png)  

(3) 사설 Git 주소를 기입합니다.  
![](./img/scm3.png)  

(4) SSH 공개키와 비밀키 생성하기  
Jenkins 컨테이너로 접근하여 ssh-keygen 명령어를 실행합니다.
```text
jenkins@0cb78e604852:/$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/var/jenkins_home/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /var/jenkins_home/.ssh/id_rsa
Your public key has been saved in /var/jenkins_home/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:I8wxUQDRQoADzB3VA2OaD8qcWrYuYVZcWY+kYPdpu38 jenkins@0cb78e604852
The key's randomart image is:
+---[RSA 3072]----+
|=.o**XB=.        |
|oo..Bo*++        |
| ..+.oo=..       |
|o oooo.o.        |
| =+  .+.S        |
|o= .   ...       |
|+..    .         |
|..      .  E     |
| ..      ..      |
+----[SHA256]-----+
```
생성된 공개키와 개인키를 확인합니다.
```text
jenkins@0cb78e604852:~/.ssh$ ls -ltih
total 12K
104850052 -rw-r--r-- 1 jenkins jenkins  574 Feb  6 07:37 id_rsa.pub
104850050 -rw------- 1 jenkins jenkins 2.6K Feb  6 07:37 id_rsa
```

(5) Jenkins에 Credentials 등록하기  
Jenkins에 Credentials를 등록할 때에는 SSH 개인키가 필요합니다. id_rsa의 내용을 복사합니다.  

(6) Jenkins에서 Manage Credentials를 통해서 하기와 같이 Credentials를 추가합니다.  
![](./img/scm4.png)  
![](./img/scm5.png)  

(7) 