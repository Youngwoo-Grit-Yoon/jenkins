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
Jenkins는 어떠한 빌드 도구도 사용할 수 있는 다수의 플러그인을 제공한다.
```text
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'make' --> (1)
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true --> (2)
            }
        }
    }
}
```
(1) sh 스텝은 make 명령어를 유발하고 오직 0 exit code를 받아야지만 파이프라인을 계속해서 진행한다. 0이 아닌 코드는
파이프라인을 실패시킬 것이다.  
(2) archiveArtifacts는 **/target/*.jar 패턴과 일치하는 파일들을 캡처하여 나중의 추출을 위해서 Jenkins 컨트롤러에 저장한다.  
### Test
```text
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                /* `make check` returns non-zero on test failures,
                * using `true` to allow the Pipeline to continue nonetheless
                */
                sh 'make check || true' --> (1)
                junit '**/target/*.xml' --> (2)
            }
        }
    }
}
```
(1) Using an inline shell conditional (sh 'make check || true') ensures that the sh step always sees a zero exit code, giving the junit step the opportunity to capture and process the test reports. Alternative approaches to this are covered in more detail in the Handling failure section below.  
(2) junit captures and associates the JUnit XML files matching the inclusion pattern (**/target/*.xml).
### Deploy
```text
pipeline {
    agent any

    stages {
        stage('Deploy') {
            when {
              expression {
                currentBuild.result == null || currentBuild.result == 'SUCCESS' --> (1)
              }
            }
            steps {
                sh 'make publish'
            }
        }
    }
}
```
(1) Accessing the currentBuild.result variable allows the Pipeline to determine if there were any test failures. In which case, the value would be UNSTABLE.