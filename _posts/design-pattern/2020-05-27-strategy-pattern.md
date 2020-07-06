---
layout: post
title: "전략 패턴 - Strategy Pattern [디자인패턴/코틀린/kotlin]"
description: "전략 패턴은 알고리즘 및 로직을 따로 정의하여 필요에 의해 사용 또는 교체 할 수 있는 패턴"
author: kimchanjung
date: 2020-05-27 28:00:00 +0900
categories: design pattern
published: true
---

# 전략 패턴 - Strategy Pattern

## 전략 패턴 이란
> 알고리즘 및 로직을 따로 정의하여 필요에 의해 사용 또는 교체 할 수 있는 패턴.

## 명령패턴과 차이점
### 명령패턴
 ```bash

 저장() {
    실행() // 실행위치, 파일명, 객체생성 등의 일련의 행위를 갭슐화 하여 간단하게 제공
 }
 ```
> 명령패턴은 어떤 행위자체(로직전체 또는 거의 대부분)를 직접 처리하지 않고 캡슐화된 메소드를 사용하는 것이지만  

### 전략패턴 
```bash
저장(){
   객체생성
   덧셈전략() // 전체 로직중에 일부로직을 캡슐화한 패턴을 사용하는 것 
   파일에저장()
}
 ```
 > 전략패턴은 어떤 행위를 처리할때 로직의 일부 전략을 캡슐화하여 이용하는 패턴이다.


## 장점
- 로직이나 알고리즘 변경시 해당 코드를 직접 변경하지 않고 이미 정의된 알고리즘을 교체하여 사용할 수 있다.
- 분기로직 제거

## 단점
- 객체의 수가 증가
- 코드 복잡도 증가
- 구현된 객체 사이의 결합도 증가


## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/strategy-pattern-class-diagram.png)


## 예제코드

### 전략을 추상화 한다 - 출근 전략
```kotlin
// 출근 전략
interface GoToWorkStrategy {
    fun goToBy(): String
}

class BySubwayStrategy : GoToWorkStrategy {
    override fun goToBy() = "지하철"
}

class ByCarStrategy : GoToWorkStrategy {
    override fun goToBy() = "자가용"
}

class DontGoToWorkStrategy : GoToWorkStrategy {
    override fun goToBy() = "출근안함"
}
```
### 전략을 추상화 한다 - 배달 전략
```kotlin
interface DeliveryStrategy {
    fun move(): String
}

class MotorCycleStrategy : DeliveryStrategy {
    override fun move() = "오토바이"
}

class BikeStrategy : DeliveryStrategy {
    override fun move() = "자전거"
}
```

### 클라이언트
```kotlin
class Rider(
        var name: String,
        private var goToWorkStrategy: GoToWorkStrategy,
        private var deliveryStrategy: DeliveryStrategy) {

    // 추후 변경이 생기면 전략을 갈아 끼운다
    fun changeGoToWorkStrategy(goToWorkStrategy: GoToWorkStrategy) {
        this.goToWorkStrategy = goToWorkStrategy
    }

    fun changeDeliveryStrategy(deliveryStrategy: DeliveryStrategy) {
        this.deliveryStrategy = deliveryStrategy
    }

    // 로직의 일부를 패턴화한 전략을 사용한다.
    fun goToWork() {
        // 아침에 일어난다
        // 샤워를 하고 옷을 입는다 
        goToWorkStrategy.goToBy() // 출근 시 이동수단 전략을 사용한다.
        // 내려서 지점 건물 까지 걸어간다.
    }

    fun delivery() {
        // 배달을 배차한다
        // 음식을 픽업한다.
        deliveryStrategy.move() // 배달시 이동수단 전략을 사용한다.
        // 고객에게 음식을 전달한다.
    }
}

private var motorCycleStrategy = MotorCycleStrateg()
private var bikeStrategy = BikeStrategy()
private var bySubwayStrategy = BySubwayStrategy()
private var byCarStrategy = ByCarStrategy()
private var dontGoToWorkStrategy =DontGoToWorkStrategy()

val rider = Rider("김찬정", bySubwayStrategy, motorCycleStrategy)
rider.goToWork() // 지하철 타고 출근
rider.goToDelivery() // 오토바이로 배달
rider.changeDeliveryStrategy(bikeStrategy) // 자전거배달 전략으로 변경 
rider.goToDelivery() // 자전거로 배달 
```
> 클라이언트는 출근 이동 수단 및 배달 이동 수단의 종류가 변경 되더라도 코드 변경에서 자유 롭다.
