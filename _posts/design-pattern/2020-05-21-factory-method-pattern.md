---
title: "팩토리 메소드 패턴 - Factory Method Pattern [디자인패턴]"
description: 팩토리 메소드 패턴 - Factory Method Pattern [디자인패턴]
author: kimchanjung
date: 2020-05-21 09:00:00 +0900
categories: design pattern
published: true
---

# 팩토리 메소드 패턴 - Factory Method Pattern 

## 팩토리 메소드 패턴 이란
> 객체 생성을 직접 하여 획득 하지 않고 객체를 생성/제공하는 클래스를 사용하여 획득하는 패턴

## 이해를 돕기위한 설명
### 팩토리 메소드 패턴 미적용
```bash
라이더생성(타입)
    if (타입 == "정규직")
        정규직라이더 라이더 = new 정규직라이더("김찬정")
    else if (타입 == "파트타임")
        파트타임라이더 라이더 = new 파트타임라이더("김찬정")
    else if (타입 == "야간타임")
        if (현재시각 > 오후9시)
            야간타임 라이더 = new 야간타임("김찬정")
        else
            println("야간타임 라이더 생성불가")
    ...

```
> 다양한 타입의 라이터객체를 생성할 경우 분기 처리가 필요하며  
> 야간타임 라이더 객체는 오후 9시 이후만 생성 할수있다(객체성성 조건 로직)

### 팩토리 메소드 패턴 적용

```bash
라이더생성(타입)
    라이더 라이더 = 라이더팩토리.라이더생성(타입)

```
> 분기처리 및 객체 생성 조건 로직을 팩토리객체로 위임하고 일관성있는 방법으로 객체를 생성한다.
> 다른 종류의 라이더클래스 추가 및 생성 로직의 변화도 소스코드 변경이 필요 없다.

 ## 장점
 - 클라이언트는 팩토리객체를 통하여 생성하기 때문에 생성하고자하는 클래스와  클라이언트간의 결합도가 낮아진다.
 - 하나의 메소드로 여러가지의 클래스의 객체를 생성할 수 있다.

 ## 단점
 - 객체가 늘어날 때마다 하위 클래스 재정의로 인한 불필요한 많은 클래스 생성 가능성이 있음

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/factory-method-pattern-class-diagram.png)

 ## 예제코드
 ### 클래스를 추상화
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

### 팩토리 메소드
```kotlin
class RiderFactory {
    fun createRider(className: String): Rider {
        return when (className) {
            "fulltime" -> FullTimeRider()
            "parttime" -> PartTimeRider()
            "nighttime" ->
                if (LocalDateTime.now().hour > 21)
                    NightTimeRider()
                else
                    throw IllegalArgumentException("오후 9시 이후만 생성 가능합니다")
            else -> throw IllegalArgumentException("생성할 수 있는 객체명이 아닙니다.")
        }
    }
}
```

### 클라이언트
```kotln
class DeliveryService {
    private var riderFactory = RiderFactory()

    fun assignDelivery(deliveryId: String) {
        val rider = riderFactory.createRider("fulltime")
        rider.delivery(deliveryId)
    }
}
```
> 라이터 타입에 따른 객체생을 구분할 필요 없이 일관성있는 메소드로 생성이 가능하다.
> 라이더 타입이 추가 되거나 야간타임라이더의 생성 가능 시간이 변경되어도 수정이 필요 없다. 


