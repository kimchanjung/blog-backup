---
layout: post
title: "책임 연쇄 패턴 - Chain Of Reponsibility Pattern [디자인패턴/코틀린/kotlin]"
description: "요청을 처리를 하나이상의 체인형태로 구성된 객체에게 위임하고 요청을 처리할 수 있는 객체를 만날 때까지 다음 객체로 처리를 위임하는 패턴"
author: kimchanjung
date: 2020-05-16 09:00:00 +0900
categories: design pattern
images: ["/post-img/design-pattern/chain-of-responsibility-pattern-class-diagram.png"]
published: true
---

# 책임 연쇄 패턴 - Chain Of Reponsibility Pattern

## 책임 연쇄 패턴이란
> 요청을 처리를 하나이상의 체인형태로 구성된 객체에게 위임하고 요청을 처리할 수 있는 객체를 만날 때까지 다음 객체로 처리를 위임하는 패턴


## 장점
- 요청의 처리 방식을 다음 객체로 위임하면서 분기처리가 없어진다.
- 요청의 처리를 layer 방식으로 나누어 처리하므로 객체간 결합도를 낮춘다.
 
## 단점
- 요청을 처리하는 객체들을 어떻게 구성하느냐에 따라 요청이 처리될 수도 아닐 수도 있다.
- 무한루프에 빠질 수 있으니 잘 고려해서 구성 해야함

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/chain-of-responsibility-pattern-class-diagram.png)
## 예제코드

### 처리객체
```kotlin
interface Rider {
    fun delivery(foodType: String): String
}

class FullTimeRider(var next: Rider? = null) : Rider {
    override fun delivery(foodType: String) =
            if (foodType == "양식") "양식배달" else next!!.delivery(foodType)
}

class PartTimeRider(var next: Rider? = null) : Rider {
    override fun delivery(foodType: String) =
            if (foodType == "한식") "한식배달" else next!!.delivery(foodType)
}

class ConnectRider(var next: Rider? = null) : Rider {
    override fun delivery(foodType: String) =
            if (foodType == "분식") "분식배달" else next!!.delivery(foodType)
}
```
> 처리할 수 없으면 다음 객체에 처리를 넘긴다.  
> 처리의 전체적인 구성은 인스턴스 생성시 어떻게 다음 객체들을 구성 했느냐에 따라 달라진다.  

### 클라이언트
```kotlin

class RiderService(private val rider: Rider) {
    fun delivery(foodType:String): String {
        // 분기 처리가 사라진다. 
        // if (foodType == "양식")
        //     val fullTimeRider = FullTimeRider()
        //     fullTimeRider.delivery()
        // else if (oodType == "한식")
        //     val partTimeRider = PartTimeRider()
        //     partTimeRider.delivery()
        // else
        // ....    
        return rider.delivery(foodType)
    }  
}

val riderService = RiderService(FullTimeRider(PartTimeRider(ConnectRider())))

val delivery = riderService.delivery("분식")
```