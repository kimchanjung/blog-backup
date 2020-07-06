---
layout: post
title: "퍼사드 패턴 - Facade Pattern [디자인패턴/코틀린/kotlin]"
description: "퍼사드 패턴은 아마도 우리가 프로그래밍 하면서 자연스럽게 사용되어지는 패턴중에 가장 흔한 패턴이 아닌가 싶다."
author: kimchanjung
date: 2020-05-20 09:00:00 +0900
categories: design pattern
images: ["/post-img/design-pattern/facade-pattern-class-diagram.png"]
published: true
---

# 퍼사드 패턴 - Facade Pattern

## 퍼사드 패턴이란
> 일련의 복잡한 로직을 간추려 별도의 클래스로 만들어 제공한다.  
> 퍼사드 패턴은 아마도 우리가 프로그래밍 하면서 자연스럽게 사용되어지는 패턴중에 가장 흔한 패턴이 아닌가 싶다.

## 이해를 돕기위한 설명
- AWS S3에 이미지를 업로드 한다고 가정 하자.
- AWS에서 제공하는 S3 라이브러리를 추가하고 사용하여 이미지를 업로드한다.

### 퍼사드 패턴 미적용
```bash
라이더서비스 {
    이미지저장(이미지) {
        AwsS3 s3 = AwsS3()
        이미지.이미지명
        이미지.이미지위치
        이미지.bytecode
        s3.setName(이미지.이미지명)
        s3.setLocation(이미지.이미지위치)
        ..
        s3.upload()
    }
}

배달서비스 {
    이미지저장(이미지) {
        AwsS3 s3 = AwsS3()
        이미지.이미지명
        이미지.이미지위치
        이미지.bytecode
        s3.setName(이미지.이미지명)
        s3.setLocation(이미지.이미지위치)
        ..
        s3.upload()
    }
}
```
> 클라이언트들은 매번 이미지를 업로드 할 때마다 AwsS3 객체 생성 및 필요한 정보를 설정 해줘야 한다.

### 퍼사드 패턴 적용
```bash
아마존S3서비스
   업로드이미지(이미지) {
        AwsS3 s3 = AwsS3()
        이미지.이미지명
        이미지.이미지위치
        이미지.bytecode
        s3.setName(이미지.이미지명)
        s3.setLocation(이미지.이미지위치)
        ..
        s3.upload()
   }

라이더서비스 {
    이미지저장(이미지) {
        아마존S3서비스.업로드이미지(이미지)
    }
}

배달서비스 {
    이미지저장(이미지) {
        아마존S3서비스.업로드이미지(이미지)
    }
}
```
> 업로드를 위한 일련의 작업들을 하나의 클래스로 만들어 클라이언트에게 제공한다.  
> 클라이언트는 세부 사용방법을 알 필요없이 "아마존S3서비스.업로드이미지(이미지)"만 사용하면 된다. 


## 장점
- 클라이언트와 서브시스템(라이브러리 및 클래스)간의 결합도가 줄어든다.
- 클라이언트는 서브시스템(라이브러리 및 클래스)의 다루기위한 정보와 행위가 줄어들거나 몰라도 된다.

## 단점
- Client가 서브시스템 내부의 클래스를 직접 사용하는 것을 막을 수 없다. Namespace를 선언하는 것이 대한이 될 수 있다. (<= 무슨말인지 솔직히 이해는 안됨)

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/facade-pattern-class-diagram.png)


## 예제코드

### 사용이 번거로운 외부라이브러리
```kotlin
class AwsS3 {
    lateinit var name: String
    lateinit var location: String
    fun upload() = true
}
```

### 퍼사드 패턴을 적용한 클래스
```kotlin
class AwsS3UploadService {
    private val awsS3 = AwsS3()
    fun upload(image: Image): Boolean {
        s3.name = image.name
        s3.location = image.fileDir
                ?.replace("/local/image", "")
                .toString()

        return s3.upload()
    }
}
```
> 아마존 s3 객체를 생성하고 업로드에 필요한 정보를 설정하고 업로드하기 까지 일련의 로직을 구현하여 upload 메소드로 제공한다.

### 클라이언트
```kotlin
class RiderService {
    private val awsS3UploadService = AwsS3UploadService()
    fun uploadRiderProfile(image: Image) = awsS3UploadService.upload(image)
}
```
> 라이더정보를 다루는 라이더서비스는 라이더 프로필을 업로드 하기위해 아마존 s3를 직접 다루지 않고 AwsS3UploadService를 사용한다.