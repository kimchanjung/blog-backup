---
layout: post
title: "[Spring] yarn 빌드 react 프로젝트를 gradle에 통합 포함하여 빌드하는 방법"
description: "[Spring] yarn 빌드 react 프로젝트를 gradle에 통합 포함하여 빌드하는 방법"
author: kimchanjung
date: 2020-06-23 18:00:00 +0900
categories: spring
published: true
---

# Yarn 빌드 React 프로젝트를 Spring 프로젝트 Gradle 빌드에 통합하여 빌드하는 방법
스프링 프로젝트를 빌드할 때 **백엔드 API + 프론트엔드 구성의 프로젝트** 빌드시 yarn을 사용하여 빌드하는 **프론트엔드 빌드를 Gradle 빌드에 포함**하여 빌드하는 방법을 알아 봅니다.

## 벡엔드 API와 프론트엔드의 일반적인 구성
보통 **Java Spring 백엔드 API와 React나 Vue**를 사용한 프론트엔드로 구성된 프로젝트에서는 **Spring 벡엔드 API 서버 프로젝트의 resources 폴더**에 빌드가 완료된 javascript **프론트엔드의 static 파일을 복사하여 포함** 시켜 빌드를 하는 것이 일반적인 방식이라면
프론트엔드 빌드 과정을 따로하지 않고 **백엔드 API 서버 빌드에 통합하는 방법**을 설명합니다.

## 적용 버전 정보
- **Java v1.8**
- **Spring-Boot v2.1.2**
- **Gradle v4.7**
- **[gradle-node-plugin v1.2.0](https://plugins.gradle.org/plugin/com.moowork.node)**

## Node 플러그인 의존성추가 
```groovy
buildscript {
    ext {
        springBootVersion = '2.1.2.RELEASE'
        .....
    }

    repositories {
        mavenCentral()
        maven { url "https://plugins.gradle.org/m2"}
    }

    dependencies {
        // node사용 할 수 있는 플러그인의 의존성을 추가합니다.
        classpath("com.moowork.gradle:gradle-node-plugin:1.2.0")
    }
}
```
> gradle-node-plugin 의존성을 추가합니다.

## build.gradle에 node 및 yarn 설치 명령을 추가
```groovy
// build.gralde
configure(nodeProjects) {
    // gradle에서 node명령을 사용하게 해주는 플러그인 사용을 추가합니다.
    apply plugin: 'com.moowork.node'

    node {
        version = '10.15.1' // 설치할 node 버전
        yarnVersion = '1.13.0' // 설치할 yarn 버전
        download = true
        distBaseUrl = 'https://nodejs.org/dist' // node를 다운받을 수 있는 주소
        workDir = file("${project.rootDir}/nodejs") // node를 설치할 폴더를 설정
        yarnWorkDir = file("${project.rootDir}/yarn") // yarn을 설치할 폴더를 설정
    }

    // yarn install 즉 package.json에 추가된 의존 모듈을 설치하는 명령을 설정
    task yarnInstallProduction(type: YarnTask) {
        args = ['install']
    }

    // node와 yarn을 설치하고 yarn install 명령을 하나의 테스크로 묶었다.
    task nodeModuleInstall {
        doLast {
            nodeSetup.execute() // node를 설치
            yarnSetup.execute() // yarn을 설치
            yarnInstallProduction.execute() // package.json에 추가된 의존 모듈을 설치(yarn install)
        }
    }
}
```
> Gradle 설정 파일에서 node와 yarn을 사용하기 위한 기본 설정을 추가합니다.

## 프론트빌드를 위한 설정을 추가합니다.
```groovy
// 백엔드 api 서버 설정 부분
project(":api-server") {
    
    dependencies {
        compile ('org.springframework.boot:spring-boot-starter-actuator')   
        ....                      
    }

    // 프론트엔드 프로젝트의 실제위치를 설정한다.
    node {
        nodeModulesDir = file("${rootDir}/front-end/")
    }

    // 프로젝트 빌드 명령을 설정한다
    // yran build 명령을 실행하는 것
    // 이렇게 되면 프론트엔드 프로젝트가 빌드된다.
    task yarnBuild(type: YarnTask) {
        args = ['run', "build"]
    }

    
    // 빌드된 프론트엔드 파일을 백엔드 API서버 resources에 복사한다.
    task copyDistToStatic(type: Copy) {
        // 빌드된 프론트엔드의 파일 위치
        from("${rootDir}/front-end/build/") 
        // 백엔드 API 서버의 static 리소스로 복사할 위치
        into("${project.projectDir}/src/main/resources/static/")
        includeEmptyDirs = true
    }

    
    // gralde 빌드시 사용하는 task 프론트엔드 빌드와 복사까지 전과정을 수행한다
    // nodejs 설치 -> yarn 설치 -> node_modules 다운로드 -> front-end source 빌드 -> resource/static 복사
    // ./gradlew ~~~~~~~:frontEndBuild <= 빌드시 task 명을 포함시켜 빌드 
    task frontEndBuild {
        doLast {
            nodeModuleInstall.execute()
            yarnBuild.execute()
            copyDistToStatic.execute()
        }
    }
}
```
> 프로젝트 빌드시 프론트엔드빌드 task를 추가하고 빌드 프로세스에 포함 시키는 설정을 추가합니다.

## 주의
**Gradle task 설정**을 추가 할때는 프론트엔드 빌드 task가 완전히 완료된 후 java 빌드가 수행되도록 되어야 합니다.   
그렇지 않으면 **프론트빌드가 완료되기 전에 java 빌드가 되기 때문에 프론트 파일이 미포함**되는 경우가 발생하기 때문에 주의 해야합니다.

## 마무리
Spring 프로젝트 Gradle 빌드에 **프론트엔드 빌드를 통합하여 빌드하는 방법**을 알아 보았습니다. 더 좋은 방법이 있으면 코멘트 남겨주세요
