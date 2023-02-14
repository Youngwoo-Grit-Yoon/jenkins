# HAProxy HTTPS(TLS) 설정하기
https://www.haproxy.com/blog/haproxy-ssl-termination/  

SSL Termination 기능을 사용하면 다음과 같은 이점이 있다.
- 인증서를 더 적은 곳에서 관리할 수 있다.
- 서버를 인증서 갱신을 목적으로 외부에 노출시킬 필요가 없다.
- 서버에서 암호화된 메시지를 처리할 필요가 없어진다.

SSL Termination은 한 마디로 구간 암호화다. 클라이언트와 로드 밸런서 사이가 트래픽 암호화 구간이고 로드 밸런서와 서버 사이는 트래픽
비암호화 구간이다.
## Enabling SSL with HAProxy
```text
frontend jenkins_server
    bind *:9090
    bind *:9443 ssl crt /etc/ssl/certs/server.pem
    default_backend jenkins_server
```
- ssl : 해당 리스너에 대해서 SSL Termination을 활성화 시킨다.
- crt : PEM 형식의 SSL 인증서 위치를 식별한다. 이때 해당 PEM 인증서는 Public Certificate 및 Private Key 데이터를 모두 포함하고 있어야 한다.
