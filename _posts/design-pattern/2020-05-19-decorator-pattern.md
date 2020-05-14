---
layout: post
title: "데코레이터 패턴 - Decorator Pattern [디자인패턴]"
description: 컴포지트 패턴 - Decorator Pattern [디자인패턴]
author: kimchanjung
date: 2020-05-19 09:00:00 +0900
categories: design pattern
published: true
---

# 데코레이터 패턴 - Decorator Pattern

## 데코레이터 패턴이란
>  데코레이터 패턴은 특정 클래스의 기본 기능에 추가 기능을 기존 클래스를 수정 하지 않고 패턴을 통하여 덧 붙이고 싶을 때 사용한다.

## 이해를 돕기위한 설명
- 개발자는 **개발**이라는 업무만 할 수 있다.
- 그런데 추가적으로 **개발업무**에 **프론트엔드개발**라는 업무을 넣고 싶다.
- 하지만 **개발업무**만 하는 개발자를 필요할 수도 있기 때문에 필요할때 만  **프론트엔드개발**라는 업무를 추가 하고 싶다.

### 상속을 통하여 기능을 추가한 클래스를 만들면 기능조합에 따른 클래스의 수가 늘어난다

```bash
개발
개발프론트엔드
개발백엔드
개발프론트엔드백엔드
개발안드로이드백엔드
개발안드로이드백엔드프론트엔드
....
```  
### 데코레이터 패턴을 적용하면
```bash
개발 프론트엔드 백엔드 안드로이드

개발프론트엔드 = 개발+프론트엔드
개발백엔드 = 개발+백엔드
개발안드로이드 = 개발+안드로이드
개발안드로이드백엔드 = 개발+안드로이드+백엔드
....
```  
> 정의된 클래스로 모든 경우의 조합이 가능하다.

## 장점
- 데코레이터 패턴은 기본적인 데이터에 추가할 기능이 다양하고 일정하지 않을 때 효율적이다.

## 단점
 - 연관 클래스들이 많이 필요하게 된다.
 - 코드복잡도가 증가하여 가독성이 떨어진다

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/decorator-pattern-class-diagram.png)

## 예제코드  

### 기본 기능을 하는 객체를 추상화
```kotlin
/**
 * 기본 기능 즉 "일한다" 정의한 추상 클래스를 만든다
 */
abstract class Employee(var jobType:String) {
    open fun showJobType() = jobType
    abstract fun working(): String
}

/**
 * 추상클래스를 상속한 라이더클래스를 만든다
 * 라이더 클래스의 기본업무는 음식배달이다.
 */
class Rider(jobType: String) : Employee(jobType) {
    override fun working() = "음식배달"
}

/**
 * 추상클래스를 상속한 개발자클래스를 만든다
 * 개발자 클래스의 기본업무는 java개발이다.
 */
class Developer(jobType: String) : Employee(jobType) {
    override fun working() = "java개발"
}
```  

### 업무를 추가할 수 있는 데코레이터 추상 클래스를 만든다

```kotlin
abstract class CompanyWorkDecorator : Employee("") {
    // Employee 클래스를 상속 할때 추상 메소드가 아닌 메소드를
    // 필요에 의해서 임의로 추상 메소드를 만들 수 도 있다. 
    abstract override fun showJobType(): String
}
```

### 데코레이터 추상 클래스를 상속하여 실제 추가에 사용할 클래스를 정의한다
```kotlin
class RiderWithRepairVehicle(private var rider: Employee) : CompanyWorkDecorator() {
    override fun showJobType() = rider.showJobType() + "|수리기사"
    override fun working() = rider.working() + "|오토바이수리"
}

class RiderWithManagement(private val rider: Employee) : CompanyWorkDecorator() {
    override fun showJobType() = rider.showJobType() + "|관리자"
    override fun working() = rider.working() + "|라이더관리업무"
}

class DeveloperWithReactJs(private var rider: Employee) : CompanyWorkDecorator() {
    override fun showJobType() = rider.showJobType() + "|프론트엔드"
    override fun working() = rider.working() + "|reactjs"
}

class DeveloperWithSpringBoot(private val rider: Employee) : CompanyWorkDecorator() {
    override fun showJobType() = rider.showJobType() + "|백엔드"
    override fun working() = rider.working() + "|springboot"
}
```
> 이 클래스들은 나중에 필요한 기능을 조합할때 덧붙여 조합하여 최종 기능을 만드는데 활용된다.

### 클라이언트
```kotlin
// 개발자 기본 객체 단순히 일한다는 기능만 있음
val developer = Developer("개발자")

// 개발자+ReactJS 개발 기능을 추가함
val developerWithReactJs = DeveloperWithReactJs(developer)

// 개발자+ReactJS+SpringBoot 개발 기능을 추가 하여 풀스택개발자를 만듬
val fullStackDeveloper = DeveloperWithSpringBoot(developerWithReactJs)

```
