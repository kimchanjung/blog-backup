---
layout: post
title: "옵저버 패턴 - Observer Pattern [디자인패턴]"
description: 옵저버 패턴 - Observer Pattern [디자인패턴]
author: kimchanjung
date: 2020-05-23 09:00:00 +0900
categories: design pattern
published: true
---

# 옵저버 패턴 - Observer Pattern 

## 옵저버 패턴이란
> 옵저버 패턴은 그야 말로 발행/구독 모델 이라고 생각하면 쉽다. spring 의 ApplicationEvent 이용한 이벤트 발행/구독이나 javascript onClick 같은 이벤트 리스너를 사용한다거나 크게는 aws sns/sqs 같은 것들도 발생/구독 모델이다.

## 장점
- 객체간 겹합도가 느슨해진다.

## 단점
- 옵저버의 실행 순서를 알 수 없다.

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/observer-pattern-class-diagram.png)


## 예제코드
> 배달상태 변경 시 발행자는 이벤트를 발행하고 구독자는 이벤트를 수신 받는다.  
> 배달완료 상태 이벤트를 수신 받으면 이메일을 발송한다.

### 발행/구독 인터페이스 정의
```kotlin
// 발행 인터페이스를 정의한다.
interface Publisher {
    fun update(deliveryStatus: String): Boolean
    fun add(subscriber: Subscriber): Boolean
    fun delete(subscriber: Subscriber): Boolean
   
}

// 구독 인터페이스를 정의한다.
interface Subscriber {
    fun onUpdate(deliveryStatus: String): Boolean
}
```

### 발행 인터페이스를 구현
```kotlin
class DeliveryStatusPublisher : Publisher {

    private val observerList = mutableListOf<Subscriber>()
    private var deliveryStatus = "WAIT"

    // 등록된 구독자들에게 이벤트발송
    // 등록된 구독자들을 순회하면서 이벤트 수신 받을 메소드들을 호출한다.
    override fun update(deliveryStatus: String): Boolean {
        this.deliveryStatus = deliveryStatus
        observerList.forEach { it.onUpdate(deliveryStatus) }
        return true
    }

    // 구독자를 추가한다
    override fun add(subscriber: Subscriber) = observerList.add(subscriber)

    // 구독자를 제거한다.
    override fun delete(subscriber: Subscriber) = observerList.remove(subscriber)
}
```
> 발행 구현체는 이벤트 발생 시 등록된 구독자들을 모두 순회하여 이벤트 변경을 수신받을 수 있는 메소드를 호출한다.

### 구독 인터페이스 구현
```kotlin
class DeliveryStatusSubscriber : Subscriber {
    var isSent = false
        private set

    // 배달 상태 변경, 배달 완료 상태라면 이메일을 발송한다.
    override fun onUpdate(deliveryStatus: String): Boolean {
        if (deliveryStatus == "COMPLETE") {
            sendEmail()
        }

        return true
    }

    private fun sendEmail() {
        isSent = true
    }
}
```
> 구독자는 이벤트 발생 시 발행자에서 onUpdate 메소드가 호출 되어 이벤트를 수신 받게 된다.
