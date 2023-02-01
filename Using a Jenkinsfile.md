## Using a Jenkinsfile
### Creating a Jenkinsfile
파이프라인은 일반적으로 하기와 같이 3개 스테이지의 연속적인 딜리버리 파이프라인을 수행한다.
```text
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```
`Jenkinsfile`은 프로젝트의 루트 디렉토리에 생성한다.
### Build
전형적으로 `Build` 스테이지는 소스 코드가 조립되고, 컴파일 되고, 패키징 되는 파이프라인 단계이다. `Jenkinsfile`은 빌드 도구인
Maven 등을 대체하는 것이 아니라 빌드, 테스트, 배포까지의 개발 단계를 바인딩 해주는 글루 레이어라고 볼 수 있다.