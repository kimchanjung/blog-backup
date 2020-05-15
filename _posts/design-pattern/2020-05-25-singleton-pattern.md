---
layout: post
title: "싱글톤 패턴 - Singleton Pattern [디자인패턴]"
description: 싱글톤 패턴 - Singleton Pattern [디자인패턴]
author: kimchanjung
date: 2020-05-25 09:00:00 +0900
categories: design pattern
published: true
---

# 싱글톤 패턴 - Singleton Pattern

## 싱글톤 패턴 이란
> 매번 새로운 객체를 생성하지 않고 오직 하나의 객체만 생성하어 사용하려고 할때 사용한다.

## 장점
- 메모리 낭비를 방지
- 전역 인스턴스이기 때문에 다른 클래스의 인스턴스들이 데이터를 공유하기 쉽다
- 객체를 여러개 생성해서 사용해야하는 상황에서 많이 사용 

## 단점
- private 생성자톨 갖고 있기 때문에 상속할 수 없다.
- 싱글톤은 테스트하기가 힘들다
- 서버환경에서는 싱글톤이 하나만 생성되는 것을 보장하지 못한다.
- 싱글톤의 사용은 전역상태톨 만톨수 있기 때문에 바람직하지 못하다

> 스프링은 싱글톤패턴은 대신해서 싱글톤을 만들 관리해주는 싱글톤레지스트리를 사용하여 단점을 해결함


## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/singleton-pattern-class-diagram.png)


## 예제코드
### kotlin object 키워드 사용 
```kotlin
object SingletonPattern {
    var name = "김찬정"
}
```
> object 키워드로 싱글톤이 제공된다. thread-safe 하고 lazy 한 초기화 가능

### 싱글톤 생성시 파라메터가 필요하다면 패턴 사용 
```kotlin
class LikeJavaSingletonPattern private constructor(private val name: String) {
    companion object {
        @Volatile
        private var instance: LikeJavaSingletonPattern? = null

        @Synchronized
        fun getInstance(param: String) = instance
                ?: LikeJavaSingletonPattern(param).also { instance = it }
    }
}
```
