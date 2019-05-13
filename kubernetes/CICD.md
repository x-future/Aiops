CI/CD

部署gitlab

docker pull registry.cn-hangzhou.aliyuncs.com/imooc/gitlab-ce:latest

部署Jenkins

did

<https://www.qikqiak.com/k8s-book/docs/37.Jenkins%20Pipeline.html>

<https://getintodevops.com/blog/the-simple-way-to-run-docker-in-docker-for-ci>

<https://container-solutions.com/running-docker-in-jenkins-in-docker/>

<https://forums.docker.com/t/using-docker-in-a-dockerized-jenkins-container/322/5>

<https://jenkins.io/doc/book/pipeline/docker/>

<https://www.qikqiak.com/k8s-book/docs/52.Prometheus%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8.html>



去掉JENKINS里面的CSRF Protection

授权策略允许匿名读请求

流水线pipeline

```groovy
#!groovy
pipeline {
    agent any
    environment {
        REPOSITORY="ssh://git@gitlab.mooc.com:2222/xuxiaomin/microservice.git"LE
        MODULE="user-edge-service"
        SCRIPT_PATH="/home/jenkins/scripts"
    }
    stages {
        stage('获取代码') {
            steps {
                echo "start fetch remote code from git:${REPOSITORY}"
                deleteDir()
                git "${REPOSITORY}"
            }
        }
        stage('代码静态检查') {
            steps {
                echo "start code check"
                
            }
        }
        stage('编译和单元测试') {
            steps {
                echo "start compile"
                sh "mvn -U -pl ${MODULE} -am clean package"
            }
        }
        stage('构建镜像') {
            steps {
                echo "start build image"
                sh "${SCRIPT_PATH}/build-images.sh ${MODULE}"
            }
        }
        stage('发布系统') {
            steps {
                echo "start deploy"
                sh "${SCRIPT_PATH}/deploy.sh ${MODULE}
                user-serviec-deployment ${MODULE}"
            }
        }
    }
}
```

Dockerfile

```shell
FROM openjdk:7-jre
MAINTAINER xx xxx@xxx.com
COPY target/user-edge-service-1.0-SNAPSHOP.jar /user-edge-service.jar
ENTRYPOINT ["java", "-jar", "/usr-edge-service.jar"]
```

build.sh

```shell
#!/usr/bin/env bash

mvn package
docker build -t user-edge-service:latest .
```

build-images.sh

```shell
#!/usr/bin/env bash
MODULE=$1
TIME=`date "+%Y%m%d%H"`
GIT_REVISON=`git log -l --pretty=format:"%h"`
IMAGE_NAME=hub.mooc.com:8080/micro-service/${MODULE}:${TIME}_${GIT_REVISON}
cd ${MODULE}
docker build -t ${IMAGE_NAME} .
cd -
docker push ${IMAGE_NAME}
echo "${IMAGE_NAME}" >> IMAGE_NAME
```

deploy.sh

```shell
#!/bin/bash

IMAGE=`cat IMAGE_NAME`
echo "update image to:${IMAGE}"
DEPLOYMENT=$1
MODEUL=$2
kubectl set image deployment/${DEPLOYMENT} ${MODULE}
```

