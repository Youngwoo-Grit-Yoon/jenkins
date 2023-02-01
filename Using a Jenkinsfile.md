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
