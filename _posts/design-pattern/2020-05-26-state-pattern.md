---
layout: post
title: "상태 패턴 - State Pattern [디자인패턴]"
description: 상태 패턴 - State Pattern [디자인패턴]
author: kimchanjung
date: 2020-05-26 09:00:00 +0900
categories: design pattern
published: true
---

# 상태 패턴 - State Pattern

## 상태 패턴 이란
> 객체가 상태에 따른 행위를 직접 구현하지 않고 상태와 행위를 구현한 별개의 상태객체에게 위임하는 패턴

## 이해를 돕기위한 설명
### 상태 패턴 미적용
```bash
다음상태변경(상태)
    if (상태 == 대기) {
        상태 = 배차완료
    } else if (상태 == 배차완료) {
        상태 = 배차완료
    } else if (상태 == 배달완료) {
        println("다음 상태 없음")
    }

```
> 상태 패턴을 적용하지 않으면 분기처리 및 기타 복잡한 알고리즘도 한꺼번에 같이 포함 된다.
새로운 로직이나 상태추가 시 클래스를 변경해야한다.

### 상태 패턴 적용
```bash
다음상태변경(상태패턴객체)
    상태패턴객체.다음상태()
```
> 상태에 따른 처리를 분기해서 처리하는 로직 대신 상태패턴을 구현한 객체의 다음상태() 메소드를 호출하는 것으로 일관성 있게 사용가능 하며 새로운상태 추가 및 로직 변경에 대해서 자유롭다.


## 장점
- 상태 변경에 따른 행위 로직을 직접 구현 하지 않아도 됨으로써 확장에 유리하다.
- 번잡한 분기로직이 제거된다.
 
## 단점
- 관련 클래스가 많아저 복잡도가 증가
 

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/state-pattern-class-diagram.png)


## 예제코드
### 배달상태를 변경하는 인터페이스 정의
```kotlin
interface DeliveryStatus {
    val name: String
    fun forward(delivery: Delivery): String
    fun backward(delivery: Delivery): String
}
```

### 개별 배달상태 인터페이스를 구현
```kotlin
// 배차대기
object Wait : DeliveryStatus {
    override val name = "배차대기"

    // 배차대기의 다음 상태는 배차완료 이므로 배차완료 상태로 변경한다.
    override fun forward(delivery: Delivery): String {
        delivery.status = Assign // 해당 배달을 매겨변수로 넘겨 받아 상태를 변경시킨다.
        return delivery.status.name
    }

    // 배차대기에서 뒤로 갈 상태가 없으므로 뒤로불가 리턴한다.
    override fun backward(delivery: Delivery) = "뒤로불가"

} 

// 배차완료
object Assign : DeliveryStatus {
    override val name = "배차완료"
    override fun forward(delivery: Delivery): String {
        delivery.status = Pickup
        return delivery.status.name
    }

    override fun backward(delivery: Delivery): String {
        delivery.status = Wait
        return delivery.status.name
    }

}

// 픽업완료
object Pickup : DeliveryStatus {
    override val name = "픽업완료"
    override fun forward(delivery: Delivery): String {
        delivery.status = Complete
        return delivery.status.name
    }

    override fun backward(delivery: Delivery): String {
        delivery.status = Assign
        return delivery.status.name
    }

}

// 전달완료 
object Complete : DeliveryStatus {
    override val name = "전달완료"
    // 전달완료는 다음 상태 가 없기 때문에 진행불가
    override fun forward(delivery: Delivery) = "진행불가"

    override fun backward(delivery: Delivery): String {
        delivery.status = Pickup
        return delivery.status.name
    }

}
```

### 클라이언트
```kotlin
class Delivery {
    var status: DeliveryStatus = Wait

    fun forwardStatus() = status.forward(this)

    fun backwardStatus() = status.backward(this)
}

val delivery = Delivery()
delivery.forwardStatus() // 배차대기 => 배차완료 변경
delivery.forwardStatus() // 배차완료 => 픽업완료 변경
deliivery.backwardStatus() // 픽업완료 => 배차완료
```
> 배달클래스는 자신의 상태변경과 로직을 자신이 가지고 있지 않고 상태패턴으로 정의된 별도의 클래스를 사용한다.  
> 이로써 배달클래스는 새로운 상태추가 및 상태변경 조건 로직의 변경으로 부터 자유롭다. 