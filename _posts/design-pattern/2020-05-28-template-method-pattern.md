---
layout: post
title: "템플릿 메소드 패턴 - Template Method Pattern [디자인패턴/코틀린/kotlin]"
description: "템플릿 메소드 패턴 - Template Method Pattern [디자인패턴/코틀린/kotlin]" 
author: kimchanjung
date: 2020-05-28 16:00:00 +0900
categories: design pattern
published: true
---

# 템플릿 메소드 패턴 - Template Method Pattern

## 템플릿 메소드 패턴 이란 
> 전체적으로는 동일하지만 부분적으로 다른 경우 중복을 최소화 하는 패턴  
> 동일한 기능을 상위 클래스에 정의하고 부분적으로 다른부분은 서브클래스에 정의하여 사용한다.  
> 쉽게 말하면 같은부분은 공통으로 사용하고 약간 다른부분만 따로 생성

## 장점
- 코드중복을 줄인다.
- 자식클래스의 역할을 줄이면서 핵심로직 관리 용이
- 객체 추가 및 확장이 쉽다.

## 단점
- 추상메소드가 많아지면 관리가 복잡
- 추상클래스와 구현클래스간의 복잡성 증대

## 주의!
- 반드시 추상클래스에서 하위클래스를의 메소드나 멤버를 호출하는 의존 관계를 가져야한다
- 하위클래스에서 추상클래스를 호출하면 안된다.
- 하위클래스를 핸들링하는 주체는 상위 클래스여야 한다. 클라이언트가 하위 클래스이 메소드를 직접 호출하여 컨트롤하도록 하면 안된다.


## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/template-method-pattern-class-diagram.png)


## 예제코드

### 공통된 골격을 추상클래스로 선언
```kotlin
abstract class Rider(var name: String,
                     var age: Int,
                     var address: String) {

    // 일련의 로직을 수행한다.
    fun work(): String {
        return goToWork() + delivery() + repairVehicle() + offWork()
    }

    // 로직이 같은 부분이므로 하위 클래스가 구현할 필요 없다. 중복제거
    // 하위 클래스가 호출하지 못하도록 제한
    private fun goToWork() = "출근"
    private fun offWork() = "퇴근"

    // 로직이 다른 메소드는 하위 클래스가 구현하도록 한다
    protected abstract fun delivery(): String
    protected abstract fun repairVehicle(): String
}
```
> 공통된 로직은 하위 클래스가 상위 클래스의 것을 사용하도록 하여 중복을 줄이고  
> 부분적으로 다른 부분만 추상메소드를 정의 하여 구현하도록 마련한다. 

### 다른 부분을 확장
```kotlin
class FullTimeRider(name: String,
                    age: Int,
                    address: String,
                    var employeeNumber: String)
    : Rider(name, age, address) {

    override fun delivery() = "오토바이배달"
    override fun repairVehicle() = "오토바이수리"
}

class PartTimeRider(name: String,
                    age: Int,
                    address: String,
                    var feePerDelivery: Int)
    : Rider(name, age, address) {

    override fun delivery() = "자전거배달"
    override fun repairVehicle() = "자전거수리"
}
```
> 공통된 부분은 상위 클래스의 메소드를 사용하고 부분적으로 다른 부분은 구현한다.
> 중요한것은 상위 클래스의 work() 메소드가 일련의 로직을 제어하여 사용하도록 되어 있고 외부에 노출되고 나머지 메소드들은 사실상 외부로 부터의 접근이 불가하도록 해야한다.

### 클라이언트
```kotlin
class DeliveryService(private val rider:Rider) {
    fun delivery() {
        rider.work()
        // 일련의 메소드를 클라이언트가 사용하지 못하도록 캡슐화한다.
        // rider.goToWork()
        // rider.delivery()
        // rider.offWork()
    }
}

val deliveryService = DeliveryService(FullTimeRider())
deliveryService.delivery()
```
> 사실상 전체 로직의 구성은 동일하나 일부 로직이 부분적으로 다른 경우에 사용하는 패턴이다. 만약에 메소드들을 외부에 노출하면 로직이 달라지기 때문에 주의 해야한다.
