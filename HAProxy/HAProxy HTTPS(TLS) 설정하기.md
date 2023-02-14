# HAProxy HTTPS(TLS) 설정하기
https://www.jenkins.io/doc/book/system-administration/reverse-proxy-configuration-with-jenkins/reverse-proxy-configuration-haproxy/

SSL Termination 기능을 사용하면 다음과 같은 이점이 있다.
- 인증서를 더 적은 곳에서 관리할 수 있다.
- 서버를 인증서 갱신을 목적으로 외부에 노출시킬 필요가 없다.
- 서버에서 암호화된 메시지를 처리할 필요가 없어진다.

SSL Termination은 한 마디로 구간 암호화다. 클라이언트와 로드 밸런서 사이가 트래픽 암호화 구간이고 로드 밸런서와 서버 사이는 트래픽
비암호화 구간이다.
## With SSL
HAProxy 2.6.7부터 SSL Termination 기능 사용을 위해서 haproxy.cfg 파일을 하기와 같이 작성하면 된다.
```text
# If you already have an haproxy.cfg file, you can probably leave the
# global and defaults section as-is, but you might need to increase the
# timeouts so that long-running CLI commands will work.

global
    maxconn 4096
    log stdout local0 debug

defaults
   log global
   option httplog
   option dontlognull
   option forwardfor
   maxconn 20
   timeout connect 5s
   timeout client 5m
   timeout server 5m

frontend http-in
  log stdout format raw local0 debug
  bind *:80
  bind *:443 ssl crt /usr/local/etc/haproxy/ssl/server.pem
  mode http
  acl prefixed-with-jenkins  path_beg /jenkins
  http-request redirect code 301 prefix /jenkins unless prefixed-with-jenkins
  redirect scheme https if !{ ssl_fc } # Redirect http requests to https
  use_backend jenkins if prefixed-with-jenkins

backend jenkins
  log stdout format raw  local0 debug
  mode http
  server jenkins1 127.0.0.1:8080 check
  http-request replace-path /jenkins(/)?(.*) /\2
  http-request set-header X-Forwarded-Port %[dst_port]
  http-request add-header X-Forwarded-Proto https if { ssl_fc }
  http-request set-header X-Forwarded-Host %[req.hdr(Host)]
```
아래는 실제로 Jenkins 구축을 위해서 반영한 파일 내용이다.
```text
# If you already have an haproxy.cfg file, you can probably leave the
# global and defaults section as-is, but you might need to increase the
# timeouts so that long-running CLI commands will work.

global
    maxconn 4096
    log stdout local0 debug

defaults
   log global
   option httplog
   option dontlognull
   option forwardfor
   maxconn 20
   timeout connect 5s
   timeout client 5m
   timeout server 5m

frontend http-in
  log stdout format raw local0 debug
  bind *:9090 ssl crt /var/lib/haproxy/server.pem
  mode http
  acl prefixed-with-jenkins  path_beg /jenkins
  http-request redirect code 301 prefix /jenkins unless prefixed-with-jenkins
  redirect scheme https if !{ ssl_fc } # Redirect http requests to https
  use_backend jenkins if prefixed-with-jenkins

backend jenkins
  log stdout format raw  local0 debug
  mode http
  server jenkins1 192.168.53.9:8080 check
  http-request replace-path /jenkins(/)?(.*) /\2
  http-request set-header X-Forwarded-Port %[dst_port]
  http-request add-header X-Forwarded-Proto https if { ssl_fc }
  http-request set-header X-Forwarded-Host %[req.hdr(Host)]
```
