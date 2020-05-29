---
layout: post
title: "커맨드 패턴 - Command Pattern [디자인패턴/코틀린/kotlin]"
description: 커맨드 패턴 - Command Pattern [디자인패턴]
author: kimchanjung
date: 2020-05-17 09:00:00 +0900
categories: design pattern
published: true
---

# 커맨드 패턴 - Command Pattern

## 커맨드 패턴이란
> 커맨드 패턴은 각각 형태가 다른 메소드를 추상화하여 클라이언트가 사용할 수 있도록 하는 패턴

## 이해를 돕기 위한 설명
### 커맨트패턴 미적용
```bash
운전(타입)
    if (타입 == 오토바이)
        오토바이 오토바이 = 오토바이()
        오토바이.시동()
        오토바이.가속핸들돌린다()
    else if (타입 == 자전거)
        자전거 자전거 = 자전거()
        자전거.페달을밟는다()
    ...        
```  

- 오토바이와 자전거가 있다 두 탈 것들을 운전하는 방법이 서로 다르다.
- 오토바이는 시동을 건다, 가속핸들을 돌린다, 자전거는 페달을 밟는다.
- 각 탈것 종류에 따라 운전하는 방법을 분기처리한다.
- 탈 것이 추가 되면 코드 수정이 필요하다.

### 커맨트 패턴 적용 
```bash
운전(배달명령)
    배달명령.탈것운전()
```  

- 여러 탈것을 운전하는 행위를 탈것운전()이라는 메소드를 추상화 하여 클라이언트에게 제공한다.
- 클라이언트는 배달명령이라는 일련의 메소드가 추상화된 배달명령의 탈것운전() 메소드만 사용한다.
- 탈 것 종류가 추가 되어도 코드 변경은 없다.

 ## 장점
 - 기존 Code를 수정하지 않고, 새 명령을 쉽게 추가할 수 있다
 - 명령의 호출자와 수신자의 의존성을 제거한다.  
 
 ## 단점
 - 명령에 대한 클래스가 늘어난다.

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/command-pattern-class-diagram.png)


## 예제코드
### 오토바이/자전거 클래스

```kotlin
class MotorCycle {
    fun start() = "오토바이시동"
    fun accelerate() = "오토바이출발"
}

class Bike {
    fun pedaling() = "자전거출발"
}
```
### 명령 메소드를 추상화
```kotlin
interface DeliveryCommand {
    fun driveVehicle(): String
}

class MotorCycleCommand(private var motorCycle: MotorCycle) : DeliveryCommand {
    override fun driveVehicle() =
            motorCycle.start() + motorCycle.accelerate()
}

class BikeCommand : DeliveryCommand {
    override fun driveVehicle() = bike.pedaling()
}
```

### 클라이언트

```kotlin
class Rider(private var deliveryCommand: DeliveryCommand) {
    fun changeDeliveryCommand(deliveryCommand: DeliveryCommand): Rider {
        this.deliveryCommand = deliveryCommand
        return this
    }

    fun delivery() = deliveryCommand.driveVehicle()
}

val motorCycleCommand = MotorCycleCommand(MotorCycle())
val bikeCommand = BikeCommand(Bike())

val rider = Rider(motorCycleCommand)

// 오토바이를 운전한다.
rider.delivery()
// 자전거로 변경한다.
rider.changeDeliveryCommand(bikeCommand)
// 자전거를 운전한다.
rider.delivery()
```
> 라이더 클라이언트는 분기문 없이 일관성있는 코드를 사용하며 로직변경이나 탈것의 추가에도 코드 수정에서 자유롭다. 

