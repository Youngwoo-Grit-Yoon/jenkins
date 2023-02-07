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
생성한 Private Registry 서버에 python:3.8 이미지를 Push 할 수 있다. 이미지 이름은 하기와 같이 Registry 정보를 포함해야
한다.
```shell
docker push localhost:7000/python:3.8
```
