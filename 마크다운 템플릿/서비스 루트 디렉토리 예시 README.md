# 한솔인티큐브 플랫폼개발2팀
## 이름
Genesys Cloud Resource API Server
## 버전
1.15
## 설명
Genesys Cloud 및 클라이언트 사이에서 시스템 리소스 CRUD 기능을 수행한다.
## 의존성
- PureCloudPlatformClientV2 `pip install PureCloudPlatformClientV2`
- FastAPI `pip install fastapi`
- uvicorn `pip install "uvicorn[standard]"`
## 실행
```shell
docker run \
-d \
--name resources-api-server \
--log-driver=json-file \
--log-opt max-size=200m \
--log-opt max-file=1 \
-p 10000:10000 \
hansolinticube/platformdev2/genesys-cloud-resources-api-server:1.15
```