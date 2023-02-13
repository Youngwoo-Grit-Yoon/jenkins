# Install HAProxy using Docker
## Description
HAProxy는 무료이며 고가용성의 오픈 소스 로드 밸런싱 애플리케이션이다. TCP와 HTTP 기반 요청을 다수의 서버 사이에서 프록시 한다.
## Write a haproxy.cfg
### Overview
https://www.haproxy.com/documentation/hapee/latest/configuration/config-sections/overview/  
HAProxy 설정 파일인 haproxy.cfg 파일은 하기와 같은 구조를 갖는다.
```text
global
 # process-level settings here

defaults
 # defaults here

frontend
 # a frontend that accepts requests from clients

backend
 # servers that fulfill the requests
```
HAProxy 공식 홈페이지에서 제공하는 Defaults 구성 내용은 하기와 같다.
```text
defaults
   mode http
   log global
   balance roundrobin
   timeout client 30s

frontend public_web_servers
   bind *:80
   default_backend public_web_servers

frontend api_servers
   bind *:8000
   default_backend api_servers

backend public_web_servers
   server s1 192.168.1.25:80
   server s2 192.168.1.26:80

backend api_servers
   server s1 192.168.1.27:8000
   server s2 192.168.1.28:8080
```
