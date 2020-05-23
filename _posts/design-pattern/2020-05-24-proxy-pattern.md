---
layout: post
title: "프록시 패턴 - Proxy Pattern [디자인패턴/코틀린/kotlin]"
description: 프록시 패턴 - Proxy Pattern [디자인패턴/코틀린/kotlin]
author: kimchanjung
date: 2020-05-24 09:00:00 +0900
categories: design pattern
published: true
---

# 프록시 패턴 - Proxy Pattern

## 프록시 패턴 이란
> 실제객체를 대신하는 객체가 로직의 흐름을 제어하여 실제 객체를 조작하는 패턴


## 장점
- 원래 객체의 기능에 부가적인 작업을 추가 할 때 용이함
- 메소드 호출이 발생하기 전까지 실제객체가 생성성되지 않는다 (메모리 이점)

## 단점
- 한단계를 거치므로 간혹 객체 응답이 느려 질 수 있댜.
- 인터페이스를 구현하므로 프록시 객체에서 구지 필요없는 메소드도 구현해야하며 코드 중복이 될 수 있다.

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/proxy-pattern-class-diagram.png)

## 예제코드
### 인터페이스 정의
```kotlin
interface Rider {
    fun delivery(): String
    fun getPersonalInfo(level: Int): String?
}
```

### 실제클래스 구현 
```kotlin
// 실제 클래스 
class FullTimeRider(var name: String) : Rider {
    override fun delivery() = "음식배달"
    override fun getPersonalInfo(level: Int) = name
}
```

### 실제객체 생성을 제어하는 프록시
```kotlin
class ProxyRider(var name: String) : Rider {
    private var fullTimeRider: FullTimeRider? = null

    override fun delivery() =
            (fullTimeRider ?: FullTimeRider(name)).delivery()

    override fun getPersonalInfo(level: Int) = name
}
```
> rider.delivery() 메소드 호출시 실제 라이더 객체가 없으면 생성, 이미 생성되어 있으면 그 객체를 사용하도록 프록시가 실제 라이터 객체의 생성에 관여 한다. 

### 라이더 정보 접근을 제어하는 프록시
```kotlin
class ProtectedProxyRider(var name: String) : Rider {
    private var fullTimeRider: FullTimeRider? = null

    private fun getRider(name: String) = fullTimeRider ?: FullTimeRider(name)

    override fun delivery() = getRider(name).delivery()

    override fun getPersonalInfo(level: Int) = if (level > 3)
        getRider(name).getPersonalInfo(level)
    else
        "접근불가"
}
```
> 라이더의 개인정보의 접근을 Level 3 초과 하는 경우에만 접근 할 수 있도록 실제 객체의 정보접근을 제어하는 프록시 

### 클라이언트
```kotlin
class DeliveryService(private val rider: Rider) {
    fun delivery() = rider.delivery()
    fun getRiderInfo() = rider.getPersonalInfo(1)
}

val deliveryService = DeliveryService(ProxyRider())
deliveryService.delivery()
```
> 배달서비스 클라이언트가 마치 실제 객체인 라이더객체를 사용하는 듯 하지만 주입된 프록시 구현체의 메소드가 호출된다.