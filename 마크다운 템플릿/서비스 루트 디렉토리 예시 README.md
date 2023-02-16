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
## Dockerfile(Optional)
```text
# Start from the official Python base image
FROM python:3.8

# This is where we'll put the requirements.txt file and the app directory
WORKDIR /code

# Copy the file with the requirements to the /code directory
COPY ./requirements.txt /code/requirements.txt

# Install the package dependencies in the requirements file
RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt

# Copy the ./app directory inside the /code directory
COPY ./app /code/app

WORKDIR /code/app

CMD ["python", "./src/main/Main.py"]
```
## Docker 이미지 빌드(Optional)
```shell
docker build --rm -t hansolinticube-platdev2-gc-resources-api-server:1.15 .
```
## 실행
```shell
docker run \
-d \
--name resources-api-server \
--log-driver=json-file \
--log-opt max-size=200m \
--log-opt max-file=1 \
-p 10000:10000 \
hansolinticube-platdev2-gc-resources-api-server:1.15
```