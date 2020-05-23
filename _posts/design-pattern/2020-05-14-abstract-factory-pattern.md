---
layout: post
title: "추상팩토리 패턴 - Abstract Factory Pattern [디자인패턴/코틀린/kotlin]"
description: 추상팩토리 패턴 - Abstract Factory Pattern
author: kimchanjung
date: 2020-05-14 09:00:00 +0900
categories: design pattern
published: true
---

# 추상팩토리 패턴 - Abstract Factory Pattern 

## 추상팩토리 패턴이란
> 추상팩토리 패턴은 어떤 연관된 클래스들을 팩토리를 통하여 일관성 있게 객체생성을 가능 가능하도록 그룹으로 묶어 제공하고 변경도 유연하도록 제공한다.

## 이해를 돕기위한 설명
### 추상팩토리 패턴 미적용 
```bash
라이더생성(라이더타입)
    if (라이터타입 == 정규직)
        정규직라이더 정규직라이더 = new 정규직라이더()
        오토바이헬멧 헬멧 = new 오토바이헬멧()
        오토바이 오토바이 = new 오토바이()
    else if (라이터타입 == 파트타임)
        파트타임라이더 파트타임라이더 = new 파트타임라이더()
        자전거헬멧 헬멧 = new 자전거헬멧()
        자전거 오토바이 = new 자전거()
    else if ()
    ...
```  

- 고용형태가 각각 다른 음식배달라이더가 있다 정규직, 파트타임
- 라이더를 생성하기 위해서 고용, 헬멧, 이동수단 등의 객체구성은 비슷하나 세부 객체는 다르다.
- 구성은 같지만 라이터의 타입이 다르면 개별로 객체생성 해야한다.
- 라이터타입이 늘어나면 분기문도 늘어나고 코드도 수정해야한다.  

### 추상팩토리 패턴 적용
```bash
라이더생성(라이더팩토리)
    라이더 = 라이더팩토리.라이더객체생성()
    헬멧 = 라이더팩토리.헬멧객체생성()
    이동수단 = 라이더팩토리.이동수단객체생성()
```  

- 분기 처리 없이 일관성있게 라이더의 업무를 지시한다
- 라이더타입이 늘어나도 코드 수정이 없다.

## 장점
- 객체생성을 팩토리에 위임 느슨한결합
- 일련의 객체 집합을 한번의 변경으로 모두 변경한다.
- 객체 집합을 생성할 때 일관성 유지(정규직라이더, 파트타임이동수단 이렇게 잘못 생성하는 경우 방지)
- 분기처리 제거
 
 
## 단점
- 객체 집합군이 늘어 날수록 관련 클래스들이 늘어나고 설계가 복잡
- 객체 집합군에 새로운 객체가 생기면 모든 팩토리를 수정해야한다.  
  정규직라이더 -> 배달, 헬멧착용, 이동, 인데 생필품배달 이라는 메소드가 생기면 
  정규직, 파트타임 전부 생필품배달 메소드를 추가해야함 

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/abstract-pattern-class-diagram.png)
## 예제 코드

### 업무가 가능한 라이더 생성에 필요한 라이더, 헬멧, 이동수단 클래스들을 추상화 한다.
#### 라이더
```kotlin
interface Rider {
    fun delivery(): String
    fun repairVehicle(): String
}

class FullTimeRider : Rider {
    override fun delivery() = "오토바이배달"
    override fun repairVehicle() = "오토바이수리"
}

class PartTimeRider : Rider {
    override fun delivery() = "자전거배달"
    override fun repairVehicle() = "자전거수리"
}
```
> 인터페이스를 구현 하도록 하여 클라이언트는 타입별 세부 구현을 알필요 없이 인터페이스만 사용한다.

#### 이동수단
```kotlin
interface Vehicle {
    fun start(): String
    fun move(): String
}

class MotorCycle : Vehicle {
    override fun start() = "시동버튼"
    override fun move() = "악셀"
}

class Bike : Vehicle {
    override fun start() = "없다"
    override fun move() = "페달"
}
```

#### 헬멧 
```kotlin
interface Helmet {
    fun wear(): String
}

class MotorCycleHelmet : Helmet {
    override fun wear() = "오토바이헬멧착용"
}

class BikeHelmet : Helmet {
    override fun wear() = "자전거헬멧착용"
}
```

### 라이더, 헬멧, 이동수단 객체를 생성하는 팩토리 클래스를 추상화

```kotlin
interface RiderFactory {
    fun getRider(): Rider
    fun getHelmet(): Helmet
    fun getVehicle(): Vehicle
}

// 팩토리 클래스는 싱글톤으로 선언한다.
object FullTimeRiderFactory : RiderFactory {
    override fun getRider() = FullTimeRider()
    override fun getHelmet() = MotorCycleHelmet()
    override fun getVehicle() = MotorCycle()
}

object PartTimeRiderFactory : RiderFactory {
    override fun getRider() = PartTimeRider()
    override fun getHelmet() = BikeHelmet()
    override fun getVehicle() = Bike()
}
```
> Client 코드에서 필요한 객체를 직접 생성하지 않고 팩토리를 통하여 생성한다.

### 라이더 객체를 사용하는 Client
```kotlin
class RiderService(private val riderFactory: RiderFactory) {
    fun work(): String {
        val rider = riderFactory.getRider()
        val helmet = riderFactory.getHelmet()
        val vehicle = riderFactory.getVehicle()
        return rider.delivery() + helmet.wear() + vehicle.move()
    }
}

// 라이더의 종류를 선택하여 팩토리를 매겨변수로 넘겨준다.
// 클라이언트가 원하는 종류를 선택하여 사용한다.
val riderService1 = RiderService(FullTimeRiderFactory)
val riderService2 = RiderService(PartTimeRiderFactory)

assertEquals("오토바이배달오토바이헬멧착용악셀", riderService1.work())
assertEquals("자전거배달자전거헬멧착용페달", riderService2.work())
```

> 라이더를 생성하여 사용하는 Client는 라이터 타입별 객체생성이 분기 처리 없이 일관성있게 사용가능한 형태로 변경되었다.