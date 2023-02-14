## OpenSSL을 이용하여 Self Signed Certificate 발급하기
서버 인증서는 공개키가 서버의 공개키가 맞음을 인증 기관(CA)의 개인키로 전자서명을 한 것이며 서버와 보안 통신을 하려는 상대방이
서버 인증서를 확인하고 그 인증서 안에 있는 공개키를 이용하여 암호화된 통신을 수행할 수 있다.
### Root CA 인증서 생성
지금부터 OpenSSL을 이용하여 Root CA의 개인키와 인증서를 만드는 방법을 설명한다.

하기 명령어를 실행하여 AES256으로 암호화된 RootCA 개인키 파일을 생성한다. 생성될 개인키의 길이는 2048 비트이다. 이때 개인키를 암호화할
암호를 입력하게 되는데 꼭 기억해야 한다.
```shell
openssl genrsa -aes256 -out RootCA.key 2048
```
CSR 파일을 생성하기 위해서 RootCA.conf 파일을 하기 예시와 같이 생성한다.
```text
[req]
default_bits             = 2048
default_md               = sha1
default_keyfile          = RootCA.key
distinguished_name       = req_distinguished_name
extensions               = ca
req_extensions           = ca
 
[ca]
basicConstraints         = critical, CA:TRUE, pathlen:0
subjectKeyIdentifier     = hash
#authorityKeyIdentifier  = keyid:always, issuer:always
keyUsage                 = keyCertSign, cRLSign
nsCertType               = sslCA, emailCA, objCA

[req_distinguished_name]
countryName                      = Country Name (2 letter code)
countryName_default              = KR
countryName_min                  = 2
countryName_max                  = 2

# 회사명 입력
organizationName                 = Organization Name (eg, company)
organizationName_default         = Hansol Inticube
 
# 부서 입력
organizationalUnitName           = Organizational Unit Name (eg, section)
organizationalUnitName_default   = PlatformDev2
 
# SSL 서비스할 domain 명 입력
commonName                       = Common Name (eg, your name or your server's hostname)
commonName_default               = Jenkins Host
commonName_max                   = 64 
```
위에서 생성한 RootCA.key 및 RootCA.conf 파일을 이용하여 CSR 파일을 생성한다.
```shell
openssl req -new -key RootCA.key -out RootCA.csr -config RootCA.conf
```
지금까지 생성된 파일들은 하기와 같다.
```text
[root@localhost openssl]# ls -ltih
total 12K
7519641 -rw-r--r-- 1 root root 1.1K Feb 14 10:28 RootCA.csr
7519643 -rw-r--r-- 1 root root 1.2K Feb 14 10:20 RootCA.conf
7519638 -rw-r--r-- 1 root root 1.8K Feb 14 10:10 RootCA.key
```
이제 10년짜리 Self Signed 인증서를 생성한다. 서명에 사용할 해시 알고리즘을 변경하려면 -sha256, -sha384, 
```shell
openssl x509 -req \
-days 3650 \
-extensions ca \
-set_serial 1 \
-in RootCA.csr \
-signkey RootCA.key \
-out RootCA.crt \
-extfile RootCA.conf
```
하기는 실제로 명령어를 실행했을 때 결과이다.
```text
[root@localhost openssl]# ls -lthi
total 12K
7519641 -rw-r--r-- 1 root root 1.1K Feb 14 10:28 RootCA.csr
7519643 -rw-r--r-- 1 root root 1.2K Feb 14 10:20 RootCA.conf
7519638 -rw-r--r-- 1 root root 1.8K Feb 14 10:10 RootCA.key
[root@localhost openssl]# openssl x509 -req \
> -days 3650 \
> -extensions ca \
> -set_serial 1 \
> -in RootCA.csr \
> -signkey RootCA.key \
> -out RootCA.crt \
> -extfile RootCA.conf
Signature ok
subject=/C=KR/O=Hansol Inticube/OU=PlatformDev2/CN=Jenkins Host
Getting Private key
Enter pass phrase for RootCA.key: --> 맨 처음에 생성한 인증서 암호키 입력
[root@localhost openssl]#
```
지금까지 생성된 파일은 하기와 같다.
```text
[root@localhost openssl]# ls -ltih
total 16K
7519634 -rw-r--r-- 1 root root 1.3K Feb 14 10:39 RootCA.crt --> RootCA 공개키
7519641 -rw-r--r-- 1 root root 1.1K Feb 14 10:28 RootCA.csr
7519643 -rw-r--r-- 1 root root 1.2K Feb 14 10:20 RootCA.conf
7519638 -rw-r--r-- 1 root root 1.8K Feb 14 10:10 RootCA.key --> RootCA 개인키
```
정상적으로 RootCA 인증서가 생성되었는지 다음 명령어를 실행하여 확인할 수 있다.
```shell
openssl x509 -text -in RootCA.crt
```
### Root CA로부터 인증받은 서버 인증서 생성
서버 개인키를 생성한다.
```shell
openssl genrsa -aes256 -out server.key 2048
```
서버 CSR 파일을 생성한다.
```shell
openssl req -new -key server.key -out server.csr
```
이제부터가 중요하다. Root CA와 관련된 인증서를 만들었으니 Root CA로부터 인증받은 서버의 인증서를 생성해야 한다.
```shell
openssl x509 -req -days 3650 -in server.csr -CA RootCA.crt -CAkey RootCA.key -CAcreateserial -out server.crt
```
하기는 실제 명령어를 실행했을 때 결과이다.
```text
[root@localhost openssl]# openssl x509 -req -days 3650 -in server.csr -CA RootCA.crt -CAkey RootCA.key -CAcreateserial -out server.crt
Signature ok
subject=/C=KR/L=Default City/O=Hansol Inticube/OU=PlatformDev2/CN=Jenkins Server
Getting CA Private Key
Enter pass phrase for RootCA.key:
[root@localhost openssl]#
```
서비 개인키 파일로부터 Pass Phrase를 제거하여 서버에서 원활하게 사용할 수 있도록 한다.
```shell
openssl rsa -in server.key -out server.key
```
하기는 실제 명령어를 실행했을 때 결과이다.
```text
[root@localhost openssl]# openssl rsa -in server.key -out server.key
Enter pass phrase for server.key:
writing RSA key
[root@localhost openssl]#
```
### Conclusion
```text
[root@localhost openssl]# ls -lih
total 32K
7519643 -rw-r--r-- 1 root root 1.2K Feb 14 10:20 RootCA.conf
7519634 -rw-r--r-- 1 root root 1.3K Feb 14 10:39 RootCA.crt --> RootCA 인증서
7519641 -rw-r--r-- 1 root root 1.1K Feb 14 10:28 RootCA.csr
7519638 -rw-r--r-- 1 root root 1.8K Feb 14 10:10 RootCA.key --> RootCA 개인키
7519647 -rw-r--r-- 1 root root   17 Feb 14 11:06 RootCA.srl
7519646 -rw-r--r-- 1 root root 1.2K Feb 14 11:06 server.crt --> 서버 인증서(공개키를 포함)
7519645 -rw-r--r-- 1 root root 1013 Feb 14 10:59 server.csr
7519644 -rw-r--r-- 1 root root 1.7K Feb 14 11:09 server.key --> 서버 개인키(비밀번호 제거)
```
### *.key, *.crt 파일을 *.pem 파일로 각각 변환하기
```shell
openssl rsa -in server.key -text > server.key.pem
```
```shell
openssl x509 -inform PEM -in server.crt > server.crt.pem
```
### *.key, *.crt 파일을 하나의 *.pem 파일로 합치기
먼저 server.pem 파일을 하나 생성한다.
```shell
touch server.pem
```
server.key 파일과 server.crt 파일을 순서대로 열어서 내용 복사 후 server.pem 파일에 붙여넣는 방식으로 추가한다.
```text
-----BEGIN RSA PRIVATE KEY-----
MIIEpgIBAAKCAQEAx3/QOJKdheXB8VfCzVtrniOIbjCHCeTZAj7XJv4W6wcfyQ/E
3+8GBQXLR2YwmMmelLvdgAcGxMKZX9T+vLRZa27EQyayotn9egEAL1mtQtvpcpSS
2hbYqhIOi1vuIzzofTYRe5lrm6s8noKyHy9fv2UcwvqYC3fnZO/WGWjaAFEkaPML
crd3l7Ya6ur4UV7zpF3//bdWOMqXIn1PZQ0CmuqaMwVNTENj7U8TNVn1RZx6UzIN
sZMes7hX6VYdtdn/qDVTDVWXGCMglb8DnALPsDhVTjuUcIP/VyekQiR766Vbvqyi
tq9np6reYGOM6hhM2B+GnlDpHz0SVchJ+S4kEQIDAQABAoIBAQCur+J5xxgb6OV8
GObAvu5rGJ4cLm3wYe0LpE7tG8yhnCle+2ne1AOnXj1zla2z+8faYO5BbuWkYOnA
... 내용 생략 ...
RYkDholU0fAi+WYx0zmInlrOmPugCvbT3NitwOB9HggnqXI16BNIn0blFKLEfzhU
jlnj/6354niPwgcPssIPvJexAoGBALQ//Foy3zR972zR5/DvNeyCVfKNQ6fwIej2
xfEv/1rlzKIaJpu3puP34dq+EDAR+P8tApOnUcHGsOxmSyVVLzhCrKbgdWyz4arx
L4V9deYzeq2evOvnwNevVCAqrc6c3NhaF8YWiGJ8XoJ0+UzZ2knnyTchENs8xTeF
EXIJBYf1AoGBAMPssvV9hWcr8NV3WzXpPKBdnpGfVqRm3zI0oanxh5aZTaRQ5gIj
3HKeR6Kg+aVKmqRE/di8hc9de8Ab8mcyjUoTPUXG+XClDFtD5MMiUgfOR1aXVM+3
ro5onEOprJV8bvJMAhEd5Iowlkp6tECLENWXPZjF7pQzTH0tS++TjGhU
-----END RSA PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
MIIDPzCCAicCCQCQUVyDPtkDVjANBgkqhkiG9w0BAQsFADBVMQswCQYDVQQGEwJL
UjEYMBYGA1UECgwPSGFuc29sIEludGljdWJlMRUwEwYDVQQLDAxQbGF0Zm9ybURl
djIxFTATBgNVBAMMDEplbmtpbnMgSG9zdDAeFw0yMzAyMTQwMjE3NTRaFw0zMzAy
MTEwMjE3NTRaMG4xCzAJBgNVBAYTAktSMRUwEwYDVQQHDAxEZWZhdWx0IENpdHkx
GDAWBgNVBAoMD0hhbnNvbCBJbnRpY3ViZTEVMBMGA1UECwwMUGxhdGZvcm1EZXYy
... 내용 생략 ...
HrO4V+lWHbXZ/6g1Uw1VlxgjIJW/A5wCz7A4VU47lHCD/1cnpEIke+ulW76sorav
Z6eq3mBjjOoYTNgfhp5Q6R89ElXISfkuJBECAwEAATANBgkqhkiG9w0BAQsFAAOC
AQEAMNS5Isi7wg1HKh14MU0kF9+4WnnBaiL7hP3QNaCB+d5Fz4m5izbCX3cioKAG
xZz+Uni0ZR7vFzqiNlUZEmOikj/CYazTmu7qQYoOYozsdssHOqgCfgZfyFsHNTla
BvuVMwkLegMsn36lO40OzbPY/LEHonbt1Ba156u/RJDxoi+LQ42L/FMNrKeW0fcH
xXOcihqerU3TmA+JD/yKbQwCoi5hcQlgMjPWsD8udbBOgF3ovbx8oDQGu2V5an42
nPYNdR6qSr7sAa4crIpD8yrHHFE5ip/ZTss0nR2pgq358uqQyetY0sj2p5XJaUWx
n33G4Z9NiEbBra/g+0gFRoxbEQ==
-----END CERTIFICATE-----
```
