---
layout: post
title: "방문자 패턴 - Visitor Pattern [디자인패턴/코틀린/kotlin]"
description: "방문자 패턴은 정의가 가장 어려운 패턴 중에 하나인 듯 하다 수많은 블로그의 방문자 패턴을 정의하는 문구를 아무리 봐도 이해가 가장 어려웠다"
author: kimchanjung
date: 2020-05-29 18:00:00 +0900 
categories: design pattern
images: ["/post-img/design-pattern/adapter-pattern-class-diagram.png"]
published: true
---

# 방문자 패턴 - Visitor Pattern 

## 방문자 패턴의 정의
> 패턴의 정의가 가장 어려운 패턴 중에 하나인 듯 하다 수많은 블로그의 방문자 패턴을 정의하는 문구를 아무리 봐도 이해가 가장 어려웠다. 설명으로 대체 한다.

## 이해를 돕기 위한 설명
1. 객체는 자기의 상태(필드)와 행위(메소드)를 가진다.
2. 객체의 행위의 구조는 같으나 세부 구현이 다른경우 일관성있게 처리 할 수 있다.

### 방문자 패턴 미사용
```bash
배달하기(타입)
    if (타입 == 정규직)
        정규직라이더.운전()
        정규직라이더.배달()
    else
        시간제라이더.운전()
        시간제라이더.배달()
    ...    
 ```  
 > 모든 라이더는 배달, 운전, 수리라는 행위를 하지만 라이더의 타입에 따라 행위의 세부 구현은 다르다. 
 > 모든 타입의 라이더의 일련의 행위를 모두 호출한다면 각각 호출 해야한다.  

### 인터페이스 사용한다면 
```bash
배달하기(라이더)
   라이더.운전()
   라이더.배달() 
```   
> 인터페이스를 타입별로 구현하면 위 처럼 가능하다. 하지만 운전(), 배달() 각각의 메소드는 모두 호출 해야한다.

### 방문자 패턴 사용
```bash
배달하기()
    for(라이더: 라이더객체리스트) // 정규직라이더, 시간제라이더 ..
        라이더.업무() // 운전하기(), 배달하기(), 수리하기()....
```
> 방문자 패턴을 사용하면 여려객체(정규직라이더, 시간제라이더) 타입을 일반화하고 거기에 여러 메소드들(운전하기, 배달하기, 퇴근하기)까지 일반화 하여 사용 할 수 있도록 해준다.

## 장점
- 클래스타입/메소드종류에 관계없이 추상화된 메소드로 일관성 있게 사용함으로써 클래스/메소드 추가나 로직 변경으로 부터 자유롭다.  

## 단점
- 관련 클래스들이 늘어남에 따라 구조가 복잡해 진다.

## 클래스 다이어그램
![class-diagram](/post-img/design-pattern/visitor-pattern-class-diagram.png)

## 예제코드

### 라이더의 행위(메소드)들을 추상화 한다.
```kotlin
// 라이더의 개별 행위 즉 배달, 운전, 수리를 업무수행이라는 메소드로 추상황 한다.
interface RiderTask {
    fun doTask(rider: Rider): String
}

// 추상화된 메소드 업무수행을 배달, 운전, 수리 구현체에서 구현한다.
class DeliveryTask : RiderTask {
    override fun doTask(rider: Rider) = rider.doTask(this)

}

class DriveVehicleTask : RiderTask {
    override fun doTask(rider: Rider) = rider.doTask(this)

}

class RepairVehicleTask : RiderTask {
    override fun doTask(rider: Rider) = rider.doTask(this)

}
```
> delivery(), driveVehivle().. 등의 각각 메소드들을 사실상 doTask라는 메소드명 1개로 사용하기 위해서 하는 작업

### 라이더 타입별로 구현한다.
```kotlin
// 라이더의 행위들을 정의 한다.
interface Rider {
    fun doTask(deliveryTask: DeliveryTask): String
    fun doTask(driveVehicleTask: DriveVehicleTask): String
    fun doTask(repairVehicleTask: RepairVehicleTask): String
}

// 라이더의 타입에 따라 각각의 행위를 구현한다.
// 행위들은 이제 fullTimeRider.delivery(),fullTimeRider.driveVehicle() 대신
// rider.doTask() 추상화 되어 수행 될 것이다.
class FullTimeRider : Rider {
    override fun doTask(deliveryTask: DeliveryTask) = "오토바이배달"
    override fun doTask(driveVehicleTask: DriveVehicleTask) = "오토바이를운전한다"
    override fun doTask(repairVehicleTask: RepairVehicleTask) = "오토바이를수리한다"
}

class PartTimeRider : Rider {
    override fun doTask(deliveryTask: DeliveryTask) = "자전거배달"
    override fun doTask(driveVehicleTask: DriveVehicleTask) = "자전거를운전한다"
    override fun doTask(repairVehicleTask: RepairVehicleTask) = "자전거를수리한다"
}
```

### 클라이언트
```kotlin
class TotalRiderTask : RiderTask {
    private val riderTaskList =
            mutableListOf(DeliveryTask(), DriveVehicleTask(), RepairVehicleTask())

    // 배달/운전/수리 모든 메소드를 수행한다.
    override fun doTask(rider: Rider) =
            riderTaskList.joinToString(separator = ""){ it.doTask(rider) }
}

val fullTimeRider = FullTimeRider()
val partTimeRider = PartTimeRider()
val totalRiderTask = TotalRiderTask()

// 정규직 라이더의 배달/운전/수리 일련의 행위를 수행
totalRiderTask.doTask(fullTimeRider)

// 시간제 라이더의 배달/운전/수리 일련의 행위를 수행
totalRiderTask.doTask(partTimeRider)
```
> doTask 메소드는 정규직/시간제 라이더 타입별, 배달/운전/수리 메소드별 처리를 구분하는 로직이 필요 없고 
추후 라이더 타입 추가 및 메소드 추가에도 코드 변경으로 부터 자유 롭다.