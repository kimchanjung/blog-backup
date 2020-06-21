---
layout: post
title: "컴포지트 패턴 - Composite Pattern [디자인패턴/코틀린/kotlin]"
description: "컴포지트 패턴 - Composite Pattern [디자인패턴/코틀린/kotlin]"
author: kimchanjung
date: 2020-05-18 09:00:00 +0900
categories: design pattern
published: true
---

# 컴포지트 패턴 - Composite Pattern

## 컴포지트 패턴이란
> 하나 또는 하나이상의 객체이거나 상관없이 하나의 객체처럼 다룰 수 있게 해주는 패턴

## 이해를 돕기위한 설명
### 컴포지트 패턴 미적용
```bash
모든라이더들배달()
    정규직라이더 정규직라이더 = new 정규직라이더()
    파트타임라이더 파트타임라이더 = new 파트타임라이더()
    배민커넥트라이더 배민커넥트라이더 = new 배민커넥트라이더()

    정규직라이더.배달()
    파트타임라이더.배달()
    배민커넥트라이더.배달()

    ...
```
> 모든 라이더타입들의 객체를 생성하고 배달 메소드를 모드 호출한다. 라이더타입이 추가될 때마다 코드는 수정 되어야 한다.  

### 컴포지트 패턴 적용
```bash
모든라이더들배달()
    for (라이더 : 라이더리스트)
        라이더.배달()
```
> 모든 라이더타입은 인터페이스를 구현하도록 하고 리스트에 포함 된다.
> 리스트를 순회 하여 배달() 메소드를 호출하도록 코드화함으로써 개별 객체생성 및 메소드 호출 코드가 제거 된다.  
> 라이더타입이 추가되어도 변경으로 부터 자유롭다.


## 장점
- 하나던 여러개던 일관성있는 방법으로 객체를 사용가능하며 클라이언트 코드가 단순해짐
- 새로운 객체를 추가하는 것이 용이함 클라이언트 코드를 변경할 필요 없음

## 단점
- 여러 종류의 객체를 일반화 시켰기 때문에 특정 객체에 제약 조건을 주거나 할 수 없다.

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/composite-pattern-class-diagram.png)


## 예제코드

### 라이더타입별 클래스를 추상화 

```kotlin
interface Rider {
    fun delivery(vehicle: String): String
}


class FullTimeRider : Rider {
    override fun delivery(vehicle: String) = "정규직" + vehicle + "배달"
}

class PartTimeRider : Rider {
    override fun delivery(vehicle: String) = "시간제"+ vehicle + "배달"
}

class ConnectRider : Rider {
    override fun delivery(vehicle: String) = "커넥트"+ vehicle + "배달"
}
```  

### 리스트로 관리되는 라이더객체들
```kotlin
class AllTypeRider(private val riders: MutableList<Rider>) : Rider {

    override fun delivery(vehicle: String): String {
        return riders.joinToString(separator = "") { it.delivery(vehicle) }
    }

    fun add(rider: Rider) = riders.add(rider)

    fun remove(rider: Rider) = riders.remove(rider)
}
```
> 클라이언트에게 마치 하나의 객체를 다루는 방식으로 제공한다  

### 클라이언트 
```kotlin
class RiderService(private val allTypeRider: AllTypeRider) {
    fun deliveryAllRiders(vehicle: String) =
            allTypeRider.delivery(vehicle)
}

val allTypeRider = AllTypeRider(mutableListOf(FullTimeRider(), PartTimeRider(), ConnectRider()))

// 새로운 라이더타입의 객체를 추가 할 수도 있다
allTypeRider.add(NewRiderType())

// 모든라이더의 배달 메소드를 호출한다.
val riderService = RiderService(allTypeRider)
val delivery = riderService.deliveryAllRiders("자전거")
```
> 클라이언트는 하나의 객체처럼 다룬다. 
