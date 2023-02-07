## Docker Private Registry 서버 구축하는 법
https://docs.docker.com/registry/deploying/#basic-configuration
### 간단하게 Docker Registry 서버 생성하기
```text
docker run -d \
  -p 5000:5000 \ --> Registry 접근 포트
  --restart=always \
  --name registry \
  -v /mnt/registry:/var/lib/registry \ --> Registry 저장소 마운트
  registry:2
```
하기 내용 복사해서 바로 실행이 가능하다.
```shell
docker run -d \
  -p 7000:5000 \
  --restart=always \
  --name private-registry \
  -v /root/jenkins/private_registry:/var/lib/registry \
  registry:2
```
