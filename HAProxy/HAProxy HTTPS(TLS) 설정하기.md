# HAProxy HTTPS(TLS) 설정하기
https://www.haproxy.com/blog/haproxy-ssl-termination/  

SSL Termination 기능을 사용하면 다음과 같은 이점이 있다.
- 인증서를 더 적은 곳에서 관리할 수 있다.
- 서버를 인증서 갱신을 목적으로 외부에 노출시킬 필요가 없다.
- 서버에서 암호화된 메시지를 처리할 필요가 없어진다.

SSL Termination은 한 마디로 구간 암호화다. 클라이언트와 로드 밸런서 사이가 트래픽 암호화 구간이고 로드 밸런서와 서버 사이는 트래픽
비암호화 구간이다.
