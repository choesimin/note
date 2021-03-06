# Freestyle Project

- Build에서 shell script 선택해서 shell 명령어 적으면 됨

## GitLab 연결

1. 'DashBoard > Manage Jenkins > Security > Manage Credentials'에서 'Add credential' click
2. kind 항목을 Username with Password로 선택하고, GitLab의 계정, 비밀번호 입력
3. '[내 project] > Configure > SourceCodeManagement'에서 'Git' check
4. URL 입력 후, Credential 선택

## Project의 Node version 변경

- gradle, maven도 비슷함

1. 'DashBoard > Manage Jenkins > Manage Plugins'에서 'NodeJS Plugin' 설치
2. (reference 문서의 설명 따라서 )원하는 version 설치
3. '[내 project] > Configure > Build Environment'의 'Provide Node & npm bin/ folder to PATH' check
4. (2번 과정에서 설치한 )Node version 선택

---

# Pipeline Project

- Scripted와 Declarative 두 가지가 있음
    - 서로 호환되지 않아서 둘 중 하나로 문법을 통일하여 작성해야 함

## Scripted

- 장점
    - 더 많은 절차적인 code를 작성 가능
        - 더 다양한 작업을 생성할 수 있음
    - program 작성과 흡사
    - 기존 pipeline 문법이라 친숙하고 이전 version과 호환 가능
    - 필요한 경우 custom한 작업 생성이 가능하기 때문에 유연성 좋음
    - 보다 복잡한 workflow 및 pipeline modeling이 가능
- 단점
    - 일반적으로 더 많은 programming이 필요
        - 복잡하고 유지보수하기 더 힘듬
    - Groovy 언어 및 환경으로 제한된 구문 검사
    - 전통적인 Jenkins model과 맞지 않음
    - 같은 작업이라면 Declarative 문법보다 잠재적으로 더 복잡
- directive
    |Directive|설명|
    |--|--|
    |node|scripted pipeline을 실행하는 Jenkins agent. 최상단 선언 필요. Jenkins master-slave 구조에서는 parameter로 master-slave 정의 가능|
    |dir|명령을 수행할 directory/folder 정의|
    |stage|pipeline의 각 단계를 얘기하며, 해당 단계에서 어떤 작업을 실행할지 선언하는 곳 (== 작업의 본문)|
    |git|Git 원격 저장소에서 project clone|
    |sh|Unix 환경에서 실행할 명령어 실행 (Windows에서는 'bat')|
    |def|Groovy 변수 혹은 함수 선언 (javascript의 var같은 존재)|
- Node & Stage 간의 관계
    - Node > Stage > Step
    - Node
        - Stage
            - Step
            - Step
        - Stage
            - Step
            - Step
            - Step
        - ...
            - ...
- example

    ```groovy
    node {
        def hello = 'Hello jojoldu'    // 변수선언
        stage ('clone') {
            git 'https://github.com/simiin/example.git'    // git clone
        }
        dir ('sample') { // clone 받은 project 안의 sample directory에서 stage 실행
            stage ('sample/execute') {
                sh './execute.sh'
            }
        }
        stage ('print') {
            print(hello) // 함수 + 변수 사용
        }
    }

    // 함수 선언 (반환 type이 없기 때문에 void로 선언, 있다면 def로 선언하면 됨)
    void print(message) {
        echo "${message}"
    }
    ```

    ```groovy
    // try-catch 예시
    node {
        stage('Example') {
            try {
                sh 'exit 1'
            } catch (exc) {
                echo 'Something failed, I should sound the klaxons!'
                throw
            }
        }
    }
    ```

## Declarative

- Scripted보다 훨씬 간단하게 작성할 수 있는 방법
- Groovy Scrip에 친숙하지 않은 사용자를 위해 최근에 도입된 declarartive programming model
- 상대적으로 작성이 쉽고 가독성이 높은 편
- CI/CD pipeline이 단순한 경우에 적합하며 아직 많은 제약사항이 따름
    - Structure만 사용할 수 있기 때문
- derective
    |Directive|설명|
    |--|--|
    |pipeline|script 방식의 'node'대신 top에 위치. Jenkinsfile이 declarative pipeline 방식으로 작성되었음을 Jenkins에게 알려줌|
    |agent|bulid를 수행할 node 및 agent를 의미. any로 지정시 build 가능한 모든 agent 중 하나를 선택하여 build를 맡김. label을 설정하여 build할 agent를 따로 선택 가능|
    |stages|1개 이상의 stage를 포함한 block. 단 하나만 선언 가능|
    |stage|scripted 방식의 stage와 동일한 개념이나, task가 아닌 steps로 구성됨|
    |steps|하나의 stage에서 수행할 작업을 명시한 block|
- example
    ```
    pipeline {    // 최상단 element로 정의되어 있어야 한다.
        agent any    // pipeline black 내 최상단에 정의되어야 하며, 말 그대로 실행할 Agent가 할당됨. 여기서 'any'는 사용 가능한 어떠한 agent로도 실행해도 된다는 걸 나타냄. 이는 pipeline 아래 위치해도 되고 각 stage block에 위치해도 됨
        options {
                skipStagesAfterUnstable()
        }
        stages {
            stage('Build') {    // stage는 pipeline의 단계를 나타냄
                steps {    // 해당 stage에서 실행할 명령을 기술하는 부분
                        sh 'make'    // 주어진 shell 명령을 수행
                }
            }
            stage('Test'){
                steps {
                    sh 'make check'
                    junit 'reports/**/*.xml'    // junit plugin을 통해 제공하는 기능. test 보고서를 집계
                }
            }
            stage('Deploy') {
                    steps {
                            sh 'make publish'
                    }
            }
        }
    }
    ```

---

# How

- Jenkins 설치
- Jenkins 실행
    - terminal에 jenkins-lts 입력하면 실행됨
    - service로 실행하면 background에 떠있게 됨
        - sudo service start jenkins
- 해당 port로 url에 입력하여 접속
- pipeline project 또는 freestyle project 생성
- script에 code 작성 후 build now

---

# Trouble Shooting

## CentOS에서 사용 시 java.nio.file.AccessDeniedException

- 추천하지 않음 : pm2나 directory의 file들은 user 기반으로 돌아가는데, root로 바꾸면 문제가 발생할 수 있음

1. Open up the this script (using VIM or other editor)

```sh
vim /etc/sysconfig/jenkins
```

2. Find this $JENKINS_USER and change to “root”

```sh
$JENKINS_USER="root"
```

3. Then change the ownership of Jenkins home, webroot and logs

```sh
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
```

4. Restart Jenkins and check the user has been changed

```sh
service jenkins restart
ps -ef | grep jenkins
```

## file 접근 권한 Error (CentOS)

- 사용자 변경
    ```sh
    chown -R svcuser.svcuser /var/lib/jenkins
    chown -R svcuser.svcuser /var/cache/jenkins/
    chown -R svcuser.svcuser /var/log/jenkins
    ```
- 실행 권한 설정 file 변경 (port 변경도 이곳에서)
    ```sh
    vi /etc/sysconfig/jenkins
    ```
    - JENKINS_USER="svcuser"
    - JENKINS_PORT="18080"
- jenkins service run (start / stop / restart)
    - 아니면 systemctl (start / stop / restart) jenkins
- 'systemctl status jenkins'로 실행 상태 확인

```sh
sudo service jenkins start
```

---

# Reference

- https://jojoldu.tistory.com/356
    - scripted
- https://jayy-h.tistory.com/32
    - declarative
- https://www.jenkins.io/doc/book/pipeline/syntax/
    - Jenkins documentation pipeline syntax
- https://cwal.tistory.com/24
    - what is Jenkins pipeline
- https://www.theserverside.com/answer/Declarative-vs-scripted-pipelines-Whats-the-difference
    - difference of scripted and declarative
- https://blog.wonizz.tk/2019/08/04/jenkins-pipeline/
    - 선언적 방식 사용 심화 예제
- https://stackoverflow.com/questions/35011699/java-nio-file-accessdeniedexception-at-jenkins-build
    - CentOS에서 실행 시 java.nio.file.AccessDeniedException
    - comment에 있음
- https://www.jenkins.io/doc/book/pipeline/
    - jenkins pipeline official documentation
- https://itraveler.tistory.com/13
    - user를 jenkins에서 다른 user로 바꾸기
        - centos jenkins AccessDeniedException Error 날 때
- http://blog.manula.org/2013/03/running-jenkins-under-different-user-in.html
    - How to run Jenkins under a different user in Linux
