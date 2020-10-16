## Playbook для  jenkins
Описание: Требуется запустить Jenkins c docker агентом

Скрипт сборки образа с docker агентом:
```
FROM jenkins/jenkins:lts
 
USER root
RUN apt-get update -qq \
    && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common 
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
RUN apt-get update  -qq \
    && apt-get install docker-ce=18.09.1~ce-0~debian -y

# RUN systemctl enable docker
# RUN usermod -aG docker jenkins

```
Запуск контейнера на один раз:
```
docker run --privileged -p 8080:8080 -p 50000:50000 jenkins_me
```
Запуск контейнера с сохранением данных предыдущей сессии
```
docker run --privileged -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins_me
```
Запуск демона на агенте:
```
sudo dockerd &
```
Проблемы:
1. Получается запустить только c максимальными привелегиями

## Сборка проекта
Ansible скрипт управления сборкой
```
pipeline {
  agent { docker { image 'python:3.7.2' } }
  stages {
    stage('build') {
      steps {
        sh 'pip install -r requirements.txt'
      }
    }
    stage('test') {
      steps {
        sh 'python test.py'
      }   
    }
  }
}
```
Скрипт с пушом и логином
```
pipeline {
    agent any
    stages {
        stage('Build image') {
            steps {
                echo 'Starting to build docker image'

                script {
                    echo 'logging in docker'
                    withCredentials([
                        usernamePassword(credentialsId: "dockerid", usernameVariable: 'USER', passwordVariable: 'PASS')
                    ]){

                    sh "docker login -u ${USER} -p ${PASS}"
                    }
                    echo 'build image'
                    def customImage = docker.build("kudddy/catcher")
                    echo 'push image'
                    customImage.push()
                }
            }
        }
    }
}
```



