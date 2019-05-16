---
layout:     post
title:      "Gitlab CI for Springboot"
subtitle:   " \"Gitlab CICD Shell Scripts for Springboot\""
date:       2019-04-20 00:00:00
author:     "Wine"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - git
    - tech
    - cicd
---


> 一、环境
1. CentOS 7  (gitlab-runner的服务器上事先安装maven,sshpass)

二、gitlab-runner
官方安装文档:https://docs.gitlab.com/runner/install/linux-manually.html
官方注册文档:https://docs.gitlab.com/runner/register/index.html

三、.gitlab-ci.yml内容 针对develop分支
```bash
#全局变量定义
variables:
  DEV_NAME: dev  		//开发服务器的用户名称
  DEV_HOST:  0.0.0.0  	//开发服务器地址
  CODE_PATH: ./Develop/SourceCode/Code/	代码路径
  MAVEN_CLI_OPTS_develop: "/mvn_config/mvn-settings.xml --batch-mode"  //编译时选择的maven路径

#阶段顺序
stages:
- before_all
- mvndepoly
- build
- deploy
- clean

#每一个阶段前面
before_script:
- ':'

#所有步骤之前
before_all:
  stage: before_all
  script:
  - ': "================      编译环境检测      ================="'
  only:
  - develop

#发布依赖包
mvn deploy dev:
  stage: mvndepoly
  # 根据不同的分支选择不同的settings.xml
  variables:
    MAVEN_CLI_OPTS: $MAVEN_CLI_OPTS_develop
  script:
  - cd ${CODE_PATH}
  - TEMP_PATH=`pwd`
  - cd springboot		//项目路径
  - pwd
  - mvn -s ${TEMP_PATH}${MAVEN_CLI_OPTS} package deploy -B -e -U -T 1C -DskipTests -Dmaven.compile.fork=true
  only:
  - develop

#结果包编译
mvn package dev:
  stage: build
  # 根据不同的分支选择不同的settings.xml
  variables:
    MAVEN_CLI_OPTS: $MAVEN_CLI_OPTS_develop
  cache:
    # 根据不同分支(branch)缓存编译包
    key: "$CI_COMMIT_REF_NAME"
    paths:
    - ${CODE_PATH}final_target
  script:
  - cd ${CODE_PATH}
  - TEMP_PATH=`pwd`
  - mkdir -p final_target
  - mvn -s ${TEMP_PATH}${MAVEN_CLI_OPTS} package -B -e -U -T 1C -DskipTests -Dmaven.compile.fork=true
  - rm -f ./final_target/*.zip
  - find ./ -name *.zip | grep -v final_target | xargs -i mv {} ./final_target/
  - find ./ -name *.zip
  only:
  - develop

#部署,运行(开发)
deploy-agri:
  stage: deploy
  variables:
    PASSWD: $DEV_PASSWD		//开发环境密码设置在runner变量下
    NAME: $DEV_NAME
    HOST: $DEV_HOST
  cache:
    key: "$CI_COMMIT_REF_NAME"
    paths:
    - ${CODE_PATH}final_target
  script:
  - ': "==============        开发环境检测       ================="'
  - find ${CODE_PATH} -name *.zip

  - ': "================          备份          ================="'
  - ': "================    停止应用,删除旧包     ================="'
  - ': "================        开始执行         ================="'
  - sshpass -p ${PASSWD} ssh -o StrictHostKeychecking=no ${NAME}@${HOST} "mkdir -p ./Releases/"
  - sshpass -p ${PASSWD} ssh -o StrictHostKeychecking=no ${NAME}@${HOST} "rm -f  ./Releases/*.zip"
  - sshpass -p ${PASSWD} scp -o StrictHostKeychecking=no ${CODE_PATH}final_target/springboot.zip ${NAME}@${HOST}:./Releases/
  - sshpass -p ${PASSWD} ssh -o StrictHostKeychecking=no ${NAME}@${HOST} "python ./Releases/deploy.py"		//备份，停止，增量，启动脚本写在deploy.py中
  - ': "=============         检测是否成功启动        =============="'
  - sshpass -p ${PASSWD} ssh -o StrictHostKeychecking=no ${NAME}@${HOST} "jps | grep -v Jps "
  only:
  - develop


after all:
  stage: clean
  script:
  - ': "======================    END    ====================="'
  - ': "===================     结果反馈？     =================="'

  only:
  - develop
```