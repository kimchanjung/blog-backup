---
layout: post
title: "플라이웨이트 패턴 - Flyweight Pattern [디자인패턴/코틀린/kotlin]"
description: "플라이웨이트 패턴은 공통으로 사용하는 클래스를 생성하는 팩토리클래스를 만들고 필요한 클래스의 인스턴스가 없는 경우 인스턴스를 새로 생성하고 이미 있는 경우는 생성해둔 인스턴스를 리턴해서 1개만 생성하여 공유해서 사용할 수 있도록 제공하는 패턴"
author: kimchanjung
date: 2020-05-22 09:00:00 +0900
categories: design pattern
images: ["/post-img/design-pattern/flyweight-pattern-class-diagram.png"]
published: true
---

# 플라이웨이트 패턴 - Flyweight Pattern 

## 플라이웨이트 패턴이란 
> 공통으로 사용하는 클래스를 생성하는 팩토리클래스를 만들고 필요한 클래스의 인스턴스가 없는 경우 인스턴스를 새로 생성하고 이미 있는 경우는 생성해둔 인스턴스를 리턴해서 1개만 생성하여 공유해서 사용할 수 있도록 제공하는 패턴

## 플라이웨이트 패턴을 사용하는 이유
`공통적으로 많이 사용하는 클래스가 있고 무수이 많은 개수가 사용`되고 있을 때 클라이언트에서 매번 `새로운 인스턴스를 생성하면 매모리 낭비`가 심한 경우가 발생하는 경우에 적용하면 유리함.

## 싱글톤 패턴과의 차이는?
싱글톤 패턴은 클래스자체가 오직 1개의 인스턴스만 허용하지만, `플라이웨이트 패턴은 싱글톤이 아닌` 클래스를 플라이웨이트 패턴의 `팩토리가 제어`하는 것이다. 인스턴스 생성의 제안을 누가 제어하느냐에 차이

## 장점
- 많은 객체를 만들때 성능을 향상시킬수 있다.
- 많은 객체를 만들때 메모리를 줄일수 있다.
 
## 단점
- 특정 인스턴스를 다르게 처리하는 것은 불가능 함

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/flyweight-pattern-class-diagram.png)

## 예제코드 - 생성 클래스가 1개 일때

```kotlin
class Rider(var center: String) {
     fun delivery() = "$center 배달"
}

class RiderFactory {
    private val riderMap = mutableMapOf<String, Rider>()

    // 해당지점 라이더 인스턴스가 없으면 새 인스턴스를 생성 후 리턴
    fun getRider(center: String) = riderMap.computeIfAbsent(center) {
        Rider(center)
    }
}

val riderFactory = RiderFactory()
val 강남라이더 = riderFactory.getRider("강남")
val 강남라이더2 = riderFactory.getRider("강남") // 이미 생성된 인스턴스를 리턴
val 관악라이더 = riderFactory.getRider("관악")
```

## 예제코드 - 비슷한 종류의 클래스가 여러개 인 경우 

### 팩토리가 생성할 클래스들을 추상화
```kotlin
interface Rider {
    var center:String
    fun delivery(): String
}

class FullTimeRider(override var center: String) : Rider {
    override fun delivery() = "$center 오토바이배달"
}

class PartTimeRider(override var center: String) : Rider {
    override fun delivery() = "$center 자전거배달"
}
```
> 클래스들을 추상화하는 이유는 여러종류의 라이더 타입 객체를 처리할 때 팩토리에서 분기 처리를 제거 하기 위함이다.

### 팩토리를 생성 
```kotlin
class RiderFactory {
    // 인스턴스 생성시 if문을 제거 하기 위해서 map활용하여 관리함 
    private val riderClasses =
            mapOf("fulltime" to FullTimeRider::class, "parttime" to PartTimeRider::class)
    // 생성된 인스턴스가 관리됨         
    private val riderMap = mutableMapOf<String, Rider>()

    // 강남지점-정규직라이더가 이미 있으면 리턴
    // 없으면 riderClasses에서 클래스원본을 가져와 리플렉션을 사용하여 새 인스턴스를 생성
    // 이 로직은 단순히 if 처리를 제거하기 위함임(패턴과는 관계 없음)
    fun getRider(center: String, type: String) =
            riderMap.computeIfAbsent(center + type) {
                riderClasses[type]!!.primaryConstructor!!.call(center)
            }
}
```

### 클라이언트
```kotlin
class RiderService {
    private val riderFactory = RiderFactory()
    fun delivery(center: String, type: String) = riderFactory
            .getRider(center, type)
            .delivery()
}

val riderService = RiderService()
riderService.delivery("강남","fulltime")
```
> 라이더의 배달을 수행하는 라이더서비스는 배달 메소드 호출 시 지점과 라이더 타입만 매개변수로 받아서 riderFactory를 사용하여 라이더 객체를 가져온다. 수많은 객체가 필요해도 riderFactory에서 적절히 관리하여 1개의 인스턴스만 유지되고 라이더 서비스는 라이더타입의 추가, 객체성성 로직의 변화로 부터 자유롭다.

