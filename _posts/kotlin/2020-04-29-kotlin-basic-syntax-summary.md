---
layout: post
title: "[kotlin/cheat sheet] 코틀린 기본 문법 요약 "
description: 코틀린 기본 문법 요약
author: kimchanjung
date: 2020-04-29 00:00:00 +0900
categories: kotlin
published: true
---
## 코틀린 기본 문법 요약

kotlin 기본 문법을 간략하게 파악 해보기 위한 요약 정리 입니다. kotlin 문법의 모든 내용을 다루지는 않지만 간략하게 훝어 보는 용도로 포스팅합니다.

## 변수 

### 변수 선언

```kotlin
val name:String = "김찬정" // 읽기만 가능
var age:String = 20 // 읽기/쓰기 가능
var address = "서울" // 타입생략이 가능

name = "홍길동" // 컴파일 오류
age = 20 // 가능
```

### Null 허용 변수와 Null을 허용하지 않는 변수 선언

```kotlin
// null 값 허용 변수 선언
var name: String? = null

// null 값을 허용하지 않는 변수에 null을 선언 할 수 없다.
var age: Int = null
```

### Null 체크 

```kotlin
val name: String? = null

// name 이 null이 아니면 length 를 반환
// java 에서  Integer nameLength = name != null? name.length() : null; 같음
val nameLength = name?.length

// name 이 null 이면 NullPointerException 발생 시킨다.
val nameLength = name!!.length

// 타입케스팅이 불가능한 경우 예외를 발생시키지 않고 null 반환한다.
val age: Int? = name as? Int?
```

## 흐름제어 
### IF 표현식

```kotlin
// String msg = msgType == 1 ? "안녕" : "잘가"; 와 같다
val msg = if (msgType == 1) "안녕" else "잘가"

// 함수 선언 시 if를 아래와 같이 사용가능 하다 
fun getMsg(msgType: Int) = if (msgType == 1) "안녕" else "잘가"
```

### WHEN 표현식
```kotlin
val inputType = 2
val msgType = "2"
fun checkType(type: Int) = if (type == 1) 1 else -1

when (inputType) {
    1 -> println("1")
    2, 3 -> println("2 or 3")
    else -> println("not")
}

// 조건에 함수를 적용할 수도 있다.
when (inputType) {
    checkType(inputType) -> println("OK")
    else -> println("NOT OK")
}

// 조건에 범위 연산자 사용도 가능하다.
// 1~100 에 포함된다면 
val result = when (inputType) {
    in 1..100 -> "1..100 OK"
    else -> "NOT OK"
}

// 분기처리할 때 확인 대상이 되는 변수를 다르게 줄 수도 있다.
when {
    inputString == "4" -> println("4 OK")
    msgType == "2" -> println("msgType = 2")
}
```

### FOR 반복문

```kotlin
val item = arrayOf(1, 2, 3)
val list = listOf(1, 2, 3)

for (index in item.indices) {
    println(index)
}

for (index in list.indices) {
    println(index)
}

// 1부터 100 까지 반복
for (i in 1..100) {
    print("$i ")
} 

// 1부터 99까지 반복
for (i in 1 until 100) {
    print("$i ")
} 

// 2 부터 10 까지 반복,  2씩 증가
for (i in 2..10 step 2) {
    print("$i ")
} 

// 10 부터 1 까지 감소
for (i in 10 downTo 1) {
    print("$i ")
} 
```

### WHILE 반복문

```kotlin
val item = Array(5) { v -> v + 1 }
var index = 0

while (index < item.size) {
    println(item[index])
    index++
}
``` 

## 함수

### 함수 선언

```kotlin
// 기본적인 함수 선언 방식
// fun basicFunc(name: String): Int <= Int는 리턴 타입이다
fun basicFunc(name: String): Int {
    return name.toInt()
}

// 한줄에 선언 하는 방식
// 한줄 선언 식에서는 리턴타입 생략 가능하다.
fun simpleFunc(name: String) = name.toInt()
```

### 기본 매개변수

```kotlin
// name: String = "김찬정" <= 매개변수에 기본 값을 선언, 함수 호출시 
// 매개변수를 주지 않으면 name은 디폴트 값인 "김찬정"으로 선언된다.
fun simpleFunc(name: String = "김찬정", age: Int): String {
    return name
} 
```


### 명명된 매개변수 - Named Parameters

```kotlin
fun simpleFunc(name: String , age: Int = 10) = name

var name = simpleFunc("김찬정", 10)
var name = simpleFunc(name = "김잔정", age = 10)
var name = simpleFunc(age = 10)
var name = simpleFunc(age = 10, name = "김찬정")
```
> 함수의 파라메터를 선언된 순서가 아닌 호출시 지정해서 줄 수 있다.

### 함수 호출 시 가변인자

```kotlin
// newList(vararg ts: T) => java에서 newList(String... name)와 같다
fun <T> newList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts)
        result.add(t)
    return result
}

val newList = newList(1, 2, 3)

// '*' 스프레드 연산자이며  item배열의 요소를 풀어서 넘긴다.
val item = Array(5) { v -> v + 1 }
val newList2 = newList(*item) 
```

### 확장함수
```kotlin
val arrayOf = arrayOf(1, 2, 3)

// Array 클래스의 length 메소드를 override 한다.
fun <T> Array<T>.length(): Int {
    return 1
}

// Array 크래스에 새로운 메소드를 추가한다.
fun <T> Array<T>.addMethod(index1: Int, index2: Int): Int {
    return 1
}


val addMethod = arrayOf.addMethod(1, 3) // 1이 리턴된다.
val length = arrayOf.length() // 1이 리턴된다.
```
> 확장함수는 특정클래스로 부터 상속받지 않고 해당클래스의 기능을 확장 한다.

### 중위함수(infix)
```kotlin
// 중의 함수 선언은 infix 키워드를 붙여 선언한다.
infix fun Int.multiply(value: Int): Int {
    return this * value
}

println(9 multiply 1) // 9 출력
println(9.multiply(1)) // 9 출력
println(3 + 2 multiply 2) // 10 출력, + 연산자 우선순위가 더 높다 
```
> 중위함수 선언 조건
> - 클래스의 멤버 함수이거 나 확장 함수이어야 한다. 
> - 매개변수가 한 개여야 한다.
> - infix 키워드로 함수가 정의되어야 한다.


### 클래스

#### 클래스 선언 - Kotlin
```kotlin
// 멤버 프로퍼티(변수)와 생성자를 동시에 선언
class SimpleClass(val name: String, var address: String, var age: Int = 41)
```
> kotlin 클래스의 기본생성자는 클래스명 바로 옆 "()" 구문이다. 기본 생성자의 매개변수 선언을 val name: String 하면 멤버 프로퍼티(변수)선언과 기본생성자 매개변수 선언을 동시에 한 것이다.  


#### 클래스 선언 - Java
```java
public class SimpleClass {
    private String name;
    private String address;
    private Integer age

    public SimpleClass(String name, String address, Integer age) {
        this.name = name;
        this.address = address
        this.age = age
    }

    public String getName() {
        return name;
    }

    public String getAddress() {
        return address;
    }

    public Integer getAge() {
        return age;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```
> Kotlin 클래스 선언이 위 Java 코드와 동일하다.

### 클래스 멤버 프로퍼티와 생성자 파라메터 동시 선언
```kotlin
// var name: String <= var/val 를 붙이면 
// 생성자 매개변수 선언과 동시에 멤버 프로퍼티(변수)를 선언한 것과 같다
class BasicClass(var name: String, var address: String) {
    // var name:String <= 한 것 과 같다
    ...
}
```

### 클래스 멤버 프로퍼티와 생성자 파라메터 각각 선언
```kotlin
// name: String <= var/val 를 붙이지 않으면 생성자 매개변수만 선언 한 것 이다.
// 그래서 멤버 프로퍼티(변수)를 꼭 선언 해야한다.
class BasicClass(name: String, address: String) {
    var name:String
    var address:String

    ...
}
```
> 위 경우에는 반드시 기본생성자의 초기화 init블럭이 있어야 한다.

### 클래스 기본생성자의 초기화 init 블럭
class BasicClass(var name: String, var address: String) 처럼 프로퍼티 선언과 생성자 매개변수 동시 선언 하지 않으면 아래 예제 처럼 멤버 프로퍼티 선언과 함께 init블럭을 통하여 멤버프로퍼티의 초기화도 해주어야 한다.
```kotlin
class BasicClass(name: String, address: String, age: Int) {
    // 프로퍼티선언과 동시에 생성자 파라메터로 값 설정이 가능하며 초기화 구문 까지 포함 된 것이다.
    var name: String = name 
    var address: String
    var age: Int

    init {
        this.address = address
        this.age = age
    }
}
```
> name 프로퍼티는 var name: String = name  선언과 동시에 초기화 구문 까지 함께 기술 하였기 때문에 init 블럭에서 초기화 구분이 제외 되었다.  
         
### 코틀린 init을  Java로 표현  
```java
public class BasicClass {
    private String name;
    private String address;
    private Integer age

    public BasicClass(String name, String address, Intger age) {
        this.name = name;
        this.address = address;
        this.age = age;
    }
}
```
> 클래스 기본생성자의 초기화 init 블럭을 Java 코드로 표현하면 위와 같다.  

### 클래스와 접근제한자

|접근 제한자| 클래스 멤버일 때 |최상위 수준으로 선언되었을 때|
|--|--|--|
|public(기본값)|어디서든 사용 가능|어디서든 사용 가능|
|internal|같은 모듈에서만 사용 가능|같은 모듈에서만 사용 가능|
|protected|서브 클래스에서만 사용 가능|해당 없음|
|private|클래스 내부에서만 사용 가능|코틀린 파일 내부에서만 사용 가능|

### 코들린의 프로퍼티란?
코틀린에서 프로퍼티란 클래스의 멤버 변수를 말하는데 코틀린에서 멤버 변수는 단순히 **var name:String** 으로 선언 되어 **변수만 선언된 것 같지만** setter/getter를 따로 선언 하지 않아도 **기본적으로 생성**되며 **변수 + setter + getter 프로퍼티**라고 부른다.  
하지만 person.name 처럼 사용하면 person.getName() 처럼 getter를 사용하지 않는 것 처럼 보이지만 사실은 person.name 값을 가져오면 내부적으로 getter를 사용 한 것이다. 


### 클래스의 프로퍼티와 override getter/setter

```kotlin
class BasicClass(name: String, address: String, age: Int) {
    var name: String 
    var address: String
        get() = "$field 특별시"
        set(address) {
            field = "$address 특별시"
        }
    var age: Int

    init {
        this.name = name 
        this.address = address
        this.age = age
    }
}

val basicClass = BasicClass("김찬정", "서울", 10)
println(basicClass.address) // 서울 특별시 로 출력 됨
basicClass.address = "서울" // 서울 특별시 로 변경됨
```
> 기본적으로 getter/setter가 제공되지만 예제 처럼 override 한 것이다.   
> basicClass.getAddress(), basicClass.setAddress("서울") 처럼 사용해야 될 것 같지만 변수를 바로 다루는 듯 사용하며 실제로 getter/setter가 동작 한다.  

#### 코들린의 public 프로퍼티의 변수는 public인가?


```kotlin
// 아래 처럼 선언하면 
class BasicClass(name:String) {
    var name:String // public 생략됨
    private var age: Int // 프로퍼티를 private로 선언하면
}
```
> 코틀린의 접근제한자는 기본적으로 멤버 프로퍼티(변수+getter+setter)에 대한접근 제한 자이다. public var name:String 은 name라는 변수는 private로 직접 접근이 불가능하며 기본으로 제공되는 getter/setter에 의해서 접근이 되는 것이고 getter/setter가 제공됨으로써 name프로퍼티는 public인 것 이다. 아래 예제를 보자 JAVA 코드로 변환 하면 실제로는 아래와 같다

```kotlin
public class BasicClass {
    private String name;
    private Integer age;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
> private var age: Int 프로퍼티를 private로 선언했기 때문에 Java 코드로 본다면  getter/setter가 제공 되지 않아 외부에서 접근 자체가 불가한 것이다.

### 생성자 오버로딩(여러개의 생성자)

```kotlin
class OverLoadingConstructor(name: String) {
    // init 블럭 그리고 constructor에 초기화 구문이 둘다 들어가 있으므로 초기화 필요 없다.
    var name: String
    lateinit var address: String
    // primitive type 프로퍼티는 사용 불가능 하기 때문에 0값으로 라도 초기화 해야한다.
    var age: Int = 0

    init {
        this.name = name
    }

    constructor(name: String, address: String, age: Int) : this(name) {
        this.name = name
        this.address = address
        this.age = age
    }
}

// 다음과 같이 생성자가 제공된다.
val overLoadingConstructor = OverLoadingConstructor("김찬정")
val overLoadingConstructor2 = OverLoadingConstructor("김찬정", "서울", 20)
```
> **lazyinit 이란** 멤버프로퍼티의 값 초기화를 지연하는 키워드이며 다음과 같은 특징이 있다.
> - var(mutable) 프로퍼티만 사용 가능
> - non-null 프로퍼티만 사용 가능
> - 커스텀 getter/setter가 없는 프로퍼티만 사용 가능
> - primitive type 프로퍼티는 사용 불가능
> - 클래스 생성자에서 사용 불가능
> - 로컬 변수로 사용 불가능


### Init 블럭 없는 생성자 오버로딩
```kotlin
class OverLoadingConstructorWithoutInit {
    var name: String
    var address: String = ""
    var age: Int = 0

    constructor(name: String) {
        this.name = name
    }

    constructor(name: String, address: String, age: Int) {
        this.name = name
        this.address = address
        this.age = age
    }
}
// 다음과 같이 생성자가 제공된다.
val overLoadingConstructor = OverLoadingConstructor("김찬정")
val overLoadingConstructor2 = OverLoadingConstructor("김찬정", "서울", 20)
```
> 클래스 선언시 클래스명 옆에 기본생성자를 선언하지 않고 constructor를 선언한다. 그렇게 되면 init 블럭도 필요없어진다.  

### 정적 팩토리 메소드
```kotlin
class PrivateDefaultConstructor private constructor() {
    lateinit var name: String

    // object 키워드로 인하여 of 메소드는 static 으로 생성된다.
    companion object {
        // 선언된 프로퍼티도 static으로 선언된다
        var address = "서울"
        fun of(name: String): PrivateDefaultConstructor {
            val instance = PrivateDefaultConstructor()
            instance.name = name
            return instance
        }
    }
}

// 정적팩토리 메소드로 객체 생성이 가능하다.  
val of = PrivateDefaultConstructor.of("김찬정")
```
> - 기본생성자를 private로 접근제한하여 외부에 제공하지 않는다.
> - companion object 사용하여 static 메소드를 생성하여 객체를 생성하는 정적 팩토리를 제공한다. 

### JAVA 코드로 보자면 아래와 같다

```java
public class PrivateDefaultConstructor {
    private String name;

    private PrivateDefaultConstructor()

    public PrivateDefaultConstructor of(String name) {
      PrivateDefaultConstructor instance =  new PrivateDefaultConstructor();
      instance.name = name
      return name;
    }
}

// 아래와 같이 정적팩토리 메소드로 객체 생성이 가능하다.
val of = PrivateDefaultConstructor.of("김찬정")
```

### JAVA 코드에서 호출을 위한 정적 팩토리 메소드 선언 방식
```kotlin
// static field에는 const 또는 @JvmField 붙인다
// static method에는 @JvmStatic 붙인다 
class PrivateDefaultConstructorForJava private constructor() {
    lateinit var name: String

    companion object {
        const val address = "서울"

        @JvmField
        val age = 20

        @JvmStatic
        fun of(name: String): PrivateDefaultConstructorForJava {
            val instance = PrivateDefaultConstructorForJava()
            instance.name = name
            return instance
        }

        @JvmStatic
        fun ofNew(name: String) = of(name)
    }
}
```
> **자바에서 코틀린 객체의 스태틱멤버를 호출 할때 PrivateDefaultConstructor.companion.address** 로 접근 해야 하기 때문에 위 예제 처럼 선언 해야 Java 코드에서 **PrivateDefaultConstructorForJava.address** 로 접근 가능하다. 

### 멤버 프로퍼티의 캡슐화
```kotlin
class BasicClass(name: String, address: String, age: Int) {
    var name: String 
        private set
    var address: String
    var age: Int

    init {
        this.name = name
        this.address = address
        this.age = age
    }
}
```
> 코틀린 프로퍼티는 기본이 public(변수자체는 private이다)이므로 값은 외부에서 참조 가능 하지만, 변경은 불가하도록 하려면 기본 setter를 private로 지정 하면 된다. (private val name:String 으로 선언하면 외부에 아예 사용 차제가 안되게 때문에)

### 클래스의 상속
```kotlin
open class Parent(name: String, age: Int) {
    var name: String
    var age: Int

    init {
        this.name = name
        this.age = age
    }

    fun isAdultParentMethod() = age > 20
    open fun getNameInEnglish(): String {
        return "kim"
    }
}

class Child(name: String, age: Int, address: String) : Parent(name, age) {
    var address: String

    init {
        this.name = name
        this.age = age
        this.address
    }

    override fun getNameInEnglish(): String {
        return "child kim"
    }
}
```
> Java 와는 다르게 open 키워드가 붙은 클래스만 상속 할 수 있다.

### 클래스 상속 - 기본생성자를 사용하지 않는 경우
```kotlin
class Child2 : Parent {
    var address: String

    constructor(name: String, age: Int, address: String) : super(name, age) {
        this.address = address
    }
}
```
> super 키워드를 통하여 부모클래스를 초기화 해야한다.

### 인터페이스 선언
```kotlin
interface ImplementInterface {
    interface ImplementInterface {
    val number: Int
    val name: String
    fun getNameInEnglish(): String
    fun isAdult(): Boolean
}
```
### 인터페이스 구현
```kotlin
class ImplementInterfaceImpl(private var age: Int) : ImplementInterface {
    override val number: Int
        get() = 1
    override val name: String = "김찬정"

    override fun getNameInEnglish(): String {
        return "kim"
    }

    override fun isAdult() = age > 19
}
```

### 추상클래스 선언과 구현
```kotlin
abstract class Car {
    val name: String = "자동차" 
    abstract fun start()
    fun stop() {}
}

class Sonata() : Car() {
    override fun start() {} 
}
```

### 추상클래스 선언 - 심화
```kotlin
abstract class AbstractClass(name: String) {
    open var name: String = ""
    var age: Int = 0
        set(value) {
            field = value
        }

    init {
        this.name = name + "이다"
    }

    fun isAdult() = age > 19
    abstract fun getNameInEnglish(): String
}
```
> - open 키워드를 붙인 프로퍼티는 구현클래스에서 override 가능
> - 추상클래스자체로 객체를 생성할 수 없지만 구현 클래스 객체 생성시 init 블럭이 수행 된다.

### 추상클래스 구현 - 심화
```kotlin
class AbstractClassImpl(name: String, age: Int, address: String) : AbstractClass(name) {
    override lateinit var name: String
    var address: String = address

    init {
        this.name = name
        this.age = ag
    }

    override fun getNameInEnglish(): String {
        return "kimchanjung"
    }
}
```
> 구현 클래스에서 추상클래스의 멤버를 직접 오버라이딩 하거나 아니면 그대로 상속 받거나, init 블럭의 초기화 순서 등등은 실제로 수행 해보면서 순서를 살펴 보아야 파악이 가능하다.

### OBJECT 키워드
```kotlin
object StaticClass {
    var name = "김찬정"
    var age = 20

    fun getNameWithEnglishName() = "$name (kimchanjung)"
    fun isAdult() = age > 19
}
println(StaticClass.name)
println(StaticClass.isAdult())
```
> Java 에서 static 멤버 변수와 메소드가 있는 클래스로 보면 된다. 객체를 런타임시 새로 생성하는 것이 아니기 때문에 또한 싱글톤이다.

### 중첩클래스와 내부클래스
```kotlin
class OuterClass {
    val outerName = "아우터네임"

    companion object {
        val staticOuterName = "정적아우터네임"
    }

    /**
     * static 키워드가 붇지 않았지만 static 멤버 클래스다
     * OuterClass.StaticNestedClass() 로 객체 생성한다.
     * Outerclass의 멤버에 접근을 할 수 없다.
     * static 멤버에는 접근이 가능함
     *
     */
    class StaticNestedClass {
        val nestedName = "내부네임"
        fun getOuterName() = println("StaticNestedClass - $nestedName $staticOuterName");
    }

    /**
     * non static 멤버 클래스
     * OuterClass().InnerClass()로 객체 생성
     * OuterClass의 non-static 멤버도 접근 가능함
     * 사실 이건 접근제어라기 보다 클래스가 static/non-static 차이에서 발생하는 자연스런 접근 제한
     */
    inner class InnerClass {
        val nestedName = "내부네임 "
        fun getOuterName() = println("InnerClass - $outerName $staticOuterName");
    }
}

// 중첩클래스의 객체 생성
val staticNestedClass = OuterClass.StaticNestedClass()
// 내부클래스의 객체 생성
val innerClass = OuterClass().InnerClass()
```
> - 중첩클래스는 외부클래스의 static 멤버로 존재하기 때문에 외부클래스.중첩클래스() 로 생성한다 (JAVA와 같다.)
> - 내부클래스는 static 멤버로 존재하는 것이 아니기 때문에 외부클래스의 인스턴스를 생성후 내부클래스 인스턴스 생성이 가능하다.

### 데이터 클래스
```kotlin
data class Entity(val id: Long, val name: String)
```
> data 클래스는 아래 메소드를 기본제공
> * equals()
> * hashCode()
> * copy()
> * toString()
> * componentsN()

### 클래스 위임
```kotlin
/*
 * by 키워드를 이용한 위임은 interface의 구현체만 위임 활 수 있다.
 * 일반 클래스를 위임하는 것은 안된다.
 */
interface UserService {
    fun findByName(name: String): String
    fun findAll(): List<String>
}

class UserServiceImpl(var name: String) : UserService {
    override fun findByName(name: String) = name;
    override fun findAll() = listOf(name, "kimchanjung")
}

/**
 * 실제적으로는 UserService를 구현하는 것이 아니라 UserServiceImpl를 상속 없이 상속 하는
 * 이라고 보면된다 open 키워드가 없어 상속 할 수 없는 UserServiceImpl 위임하여
 * 상속의 효과를 누린다.
 * 또 하나의 장점은 모든 메소드를 구현 할 필요가 없고 필요한 메소드만 오버라이드 하면된다
 */
class ImplByUserService(private val us : UserService) : UserService by us {
    override fun findAll() = listOf(name, "kimchanjung", "mogomezwai")
}
```
> 클래스위임은 상속 불가능한 클래스를 상속과 같은 효과를 누리는 동시에 필요한 메소드만 오버라이드 가능한 장점이 있다.

### 클래스 위임 장점의 예 - 인터페이스 구현
```kotlin
class NewList<T>(override val size: Int, init: (index: Int) -> T) : MutableList<T> {
    private val list = ArrayList<T>(size)

    init {
        repeat(size) { index -> list.add(init(index)) }
    }

    override fun contains(element: T): Boolean {
        TODO("Not yet implemented")
    }

    override fun containsAll(elements: Collection<T>): Boolean {
        TODO("Not yet implemented")
    }

    override fun get(index: Int): T {
        return list.get(index)
    }

    ........
}
```
> 일반적인 방식의 인터페이스구현은  MutableList interface 구현체 모든 메소드를 구현해야한다.  
 

### 클래스 위임 장점의 예 - 클래스 위임을 사용
```kotlin
class NewList<T>(override val size: Int, private val ml: MutableList<T> = mutableListOf(), init: (index: Int) -> T) : MutableList<T> by ml {

    init {
        repeat(size) { index -> ml.add(init(index)) }
    }

    override fun add(element: T): Boolean {
        return ml.add(element)
    }
}
```
> 위임을 사용한 경우는 모든 메소드를 오버라이드 할 필요 없다.

### ENUM 클래스 선언
```kotlin
enum class EnumClass(var number: Int, var desc: String) {
    WAIT(1, "대기"),
    CONSIGN(2, "배차완료중"),
    COMPLETE_PICKUP(3, "픽업완료"),
    COMPLETE_DELIVERY(4, "전달완료");

    fun getStatus() = "$desc($number)"
}
```
> 기본생성자에 반드시 var number: Int var, val 를 붙여 멤버선언/매개변수 동시에 선언해야한다.  

### ENUM 클래스를 WHEN 에서 활용
```kotlin
fun selectStatus(status: EnumClass) = when (status) {
    EnumClass.COMPLETE_DELIVERY -> "COMPLETE_DELIVERY"
    EnumClass.CONSIGN, EnumClass.COMPLETE_PICKUP -> "CONSIGN or COMPLETE_PICKUP"
    else -> "NOT"
}

fun selectStatus(status1: EnumClass, status2: EnumClass) = when (setOf(status1, status2)) {
    setOf(EnumClass.COMPLETE_DELIVERY) -> "COMPLETE_DELIVERY"
    setOf(EnumClass.CONSIGN, EnumClass.COMPLETE_PICKUP) -> "CONSIGN or COMPLETE_PICKUP"
    else -> "NOT"
}
```

### Sealed 클래스 
- Sealed 클래스는 Enum 클래스의 확장입니다. 
- Enum클래스를 사용할 때 불리한 점을 보완 할 수 있습니다.  

Sealed 클래스의 장점을 설명 하기 위하여 어떤 값을 상수나 오브젝트 형태로 사용하는 경우, Enum 클래스를 사용하는 경우 그리고 Sealed 클래스를 사용하는 경우를 예로 들어 설명합니다. 

#### 일반 값 형태로 사용 하는 경우
```kotlin
object DeliveryStatus {
    val WAIT = 1
    val CONSIGN = 2
    val PICKUP = 3
    val COMPLETE = 4 // 추후에 추가한 상태값
}


fun selectDeliveryStatus(deliveryStatus: Int): String {
    return when (deliveryStatus) {
        WAIT -> "대기($deliveryStatus)"
        CONSIGN -> "배차($deliveryStatus)"
        PICKUP -> "픽업($deliveryStatus)"
        /**
         *  DeliveryStatus 에 COMPLETE를 추가 했으면 이 라인도 당연히 추가 했어야 하지만
         *  추가 하지 않아도 컴파일 에러는 없기 때문에 개발자가 빼먹어 런타임에 의도치 않은 결과를 초래할 수 있다.
         */
        // COMPLETE -> deliveryStatus
        else -> "오류(0)"
    }
}
```
> 보통 특정 상태 값을 선언할 때는 Enum 클래스를 사용하는 것이 보통이나 일반적인 형태로 선언 하고 이것을 사용 하는 코드에서 발생 할 수 있는 문제점은 다음과 같다
> - DeliveryStatus를 사용하는 비즈니스 로직 selectDeliveryStatus에서 새로 추가한 상태값 COMPLETE에 대한 처리를 해주지 않아도 컴파일이 된다.
>  - 이런경우 런타임 시 COMPLETE는 else 로 처리 되게 됨으로 개발자가 상태 값만 추가하고 비즈니스 로직처리를 빼먹는 사태가 발 생 할 수 있다.
> - DeliveryStatus 상태가 추가 될 때 마다 비즈니스로직에 관련 처리를 계속적으로 추가 해주어야한다.
> - when 구문에서도 else 구문 추가가 필수 적이다, 그 이유는 selectDeliveryStatus(DeliveryStatus.PICKUP) 로 메소드를 호출 하는 경우 when구문 입장에서는 int 값에 어떤 값이 들어올지 미리 알 수 없기 때문에 else 구문을 추가 하지 않으면 컴파일 오류가 발생한다.  

#### ENUM 클래스를 사용 하는 경우
```kotlin
enum class EnumDeliveryStatus(val code: Int, val codeName: String) {
    WAIT(1, "대기"),
    CONSIGN(2, "배차"),
    PICKUP(3, "픽업"),

    /**
     * COMPLETE를 나중에 추가 했을 경우에 selectEnumDeliveryStatus에
     * EnumDeliveryStatus.COMPLETE -> deliveryStatus.code 이 구문을 추가 해주지 않으면
     * 컴파일 오류 발생한다.
     */
    COMPLETE(4, "완료");

    fun getCodeWithName() = "$codeName($code)"
}

/**
 * EnumDeliveryStatus 선언 당시 값이 명확히 정해 지므로 when 에서 else 처리도 불필요 해진다.
 */
fun selectEnumDeliveryStatus(deliveryStatus: EnumDeliveryStatus): String {
    return when (deliveryStatus) {
        EnumDeliveryStatus.WAIT -> "${deliveryStatus.codeName.toString()}(${deliveryStatus.code.toString()})"
        EnumDeliveryStatus.CONSIGN -> "${deliveryStatus.codeName.toString()}(${deliveryStatus.code.toString()})"
        EnumDeliveryStatus.PICKUP -> "${deliveryStatus.codeName.toString()}(${deliveryStatus.code.toString()})"
        EnumDeliveryStatus.COMPLETE -> "${deliveryStatus.codeName.toString()}(${deliveryStatus.code.toString()})"
    }
}
```
> 일반적으로 Enum 클래스를 사용 했을 경우을 살펴 보자
> - COMPLETE 상태가 추후에 추가되었을 경우 EnumDeliveryStatus 클래스를 사용하는 비즈니스로직에서
  COMPLETE 상태 처리에 대한 구문이 추가 되지 않으면 컴파일 오류가 발생하므로 개발자가 실수로 빠트리는 일은 없다.
> - Enum 클래스의 값은 선언 당시에 완전히 정해 지므로 런타임시 어떤 값이 들어올지 모르는 상황이 아니므로
  when 처리시 else 구문이 필요없게 된다.

#### Sealed 클래스를 사용 하는 경우
```kotlin
/**
 * sealed class는 enum 클래스의 확장형이라고 설명하고 있는데
 * Enum 클래스는 아래와 같이 멤퍼 프로퍼티나, 메소드를 다르게 줄 수 없지만 sealed 클래스는 가능하기 때문이다.
 */
sealed class SealedDeliveryStatus {
    class WAIT(var code: Int, var codeName: String) : SealedDeliveryStatus()
    class CONSIGN(var code: Int, var codeName: String, val riderName: String) : SealedDeliveryStatus()
    class PICKUP(var code: Int, var codeName: String) : SealedDeliveryStatus()
    class COMPLETE(var code: Int, var codeName: String, var completeTime: String) : SealedDeliveryStatus(){
        fun getCodeWithCompleteTime() = "$code $completeTime"
    }
}

/**
 * 각 상태에 맞는 멤버프로퍼티/메소드를 다르게 리턴 하도록 처리 가능하다.
 */
fun selectSealedDeliveryStatus(deliveryStatus: SealedDeliveryStatus): String {
    return when (deliveryStatus) {
        is WAIT -> deliveryStatus.codeName
        is CONSIGN -> deliveryStatus.riderName
        is PICKUP -> deliveryStatus.code.toString()
        is COMPLETE -> deliveryStatus.getCodeWithCompleteTime()
    }
}
```

## Collections
### 배열
```kotlin
val arr = arrayOfNulls<Int>(10)
val initArr = arrayOf(1, 2, 3)
val initDiffTypeArr = arrayOf(1, "2", 3L)
val initIntArr = intArrayOf(1, 2, 3)
val initConstructorArr = Array(10) { 1 }
val initLambdaArr = Array(10) { v -> v + 1 }
val initWithIncrement = (1..10).toList().toTypedArray()
val initWithIncrementByStep = (1..10).step(2).toList().toTypedArray()
val initWithIncrementByStep2 = IntRange(1, 10).step(2).toList().toTypedArray()     
```

### List
```kotlin
// 불변 List 초기화 이후 수정 삭제 불가
val listOf = listOf(1, 2, 3)
val list = List(3) { v -> v + 1 }

// 수정 삭제 추가 가능
val emptyMutableList = mutableListOf<Int>()
val mutableListOf = mutableListOf(1, 2, 3)
val mutableList = MutableList(2) { v -> v + 1 }

listOf.get(0)
listOf[0]
mutableListOf.add(3, 4)
mutableListOf.remove(1)
// even [2], odd [1,3] 각각 리스트를 구분 해서 반환함
val (even, odd) = listOf.partition { it % 2 == 0 }
```

### Map
```kotlin
// 불변
val mapOf = mapOf("a" to 1, "b" to 2)
// 변경 가능
val emptyMutableMap = mutableMapOf<String, Int>()

mapOf.get("a")
mapOf["a"]
emptyMutableMap.put("c", 3)
```

## 람다식
### 람다식 선언
```kotlin
// 기본 함수 선언 방식
fun sum(a: Int, b: Int): Int {
    return a + b
}

// 기본 함수 선언 방식을 람다식으로 표현
val sumLambda: (Int, Int) -> Int = { a, b -> a + b }

// 코틀린 함수 선언 방식
fun sum2(a: Int, b: Int) = a + b

// 코틀린 함수 선언 방식을 람다식으로 표현
val sum2Lambda = { a: Int, b: Int -> a + b }
```

### 콜렉션 람다식
```kotlin
val listOf = List(10) { v -> v + 1 }

val newList = listOf.filter { it > 2 }
                .map { it + 10 }
```

## ETC

### 연산자 오버로딩
```kotlin
operator fun Int.plus(b: String) = "$this$b"
println(10 + "2") // 102 출력
```
> + 연산자를 overloading 할 수 있다, overriding 아닌 overloading 이다!!

### 연산자 확장
```kotlin
class Position(var a: Int, var b: Int) {
    operator fun plus(position: Position): Position {
        return Position(a + position.a, b + position.b)
    }
}

val position = Position(1, 2) + Position(3, 4)
println(position.a) // 4 출력
```
> 클레스에 연산자를 확장하여 사용 할 수 있다.

### Collection get/set 확장
```kotlin
class Position(var a: Int, var b: Int) {
    operator fun set(position: Int, value: Int) {
        when (position) {
            0 -> a = value
            1 -> b = value
            else -> throw IndexOutOfBoundsException("error")
        }
    }

    operator fun get(position: Int): Int = when (position) {
        0 -> a
        1 -> b
        else -> throw IndexOutOfBoundsException("error")
    }
}
val position = Position(1,2)
position[1] = 10
println(position[1]) // 10 출력
```
> 클래스에 콜렉션의 get/set을 확장 할 수 있다.


###
### Observable
```kotlin
// 
var name: String by observable("김찬정") { property, oldValue, newValue -> println("newValue") }
name = "김찬정님"
```
> name 값이 변할 때 println 호출 된다.

### Vetoable
```kotlin
var age: Int by vetoable(0) { property, oldValue, newValue -> newValue > oldValue }

age = 10
assertEquals(10, age)
age = 5
assertEquals(10, age)
age = 20
assertEquals(20, age)
```
> age가 변경 될 때 마다 수행 되는데 true 일 때만 값이 변경 된다.

### Higher Order Function
Higher Order Function은 매개변수로 다른 함수나 람다식을 인자로 받는 함수를 말한다

```kotlin
fun calculator(a: Int, b: Int, exec: (value1: Int, value2: Int) -> Int): Int {
    return exec(a, b)
}

// 간략 하게 선언 가능
fun calculator2(exec: () -> Int) = exec()
```

### Higher Order Function의 inline/noline
```kotlin
inline fun calculator3(a: Int, exec: (value: Int) -> Int, noinline exec2: (value: Int) -> Int): Int {
    return exec(a) + exec2(a)
}
```
> - Higher Order 함수는 사용시(호출)에 파라메터로 전달되는 함수가
사실상 new 해서 새로운 객체가 매번 생기는 방식으로 동작한다 비효울 적이다
> - 이런 경우 inline 키워드를 사용하면 파라메터로 전달되는 모든 함수가 매번 new 생성 Higher Order 함수 내부에 있게 된다. 결론 적으로 매번 새로운 객체가 생성되는 것이 아니게 된다.
> - inline선언해서 모든 파라메터 함수가  매번 new 하지 않게 되었지만
파라메터로 전달되는 함수 앞에 noinline을 선언하면  new 해서 생성되게 된다.

### 예외
```kotlin
// try catch 블럭이 아니라 마치 if 처럼 동작한다
fun simpleFunction(value: String): Int? {
    val result: Int? = try {
        parseInt(value)
    } catch (e: NumberFormatException) {
        null
    }

    return result
}

// 변수에 할당 하지 않고 바로 반환
fun simpleFunction(value: String): Int? {
    return try {
        parseInt(value)
    } catch (e: NumberFormatException) {
        null
    }
}

// name이 null 이면 예외 발생 시킨다.
fun simpleFunction(name: String?): String {
    return name ?: throw IllegalArgumentException("이름을 입력 하세요")
}
```
> - 코틀린 예외는 cheked와 unchecked 구문없다. 모든 예외는 unchecked다
> - 예외 처리 블럭이 if, when 처 표현식 처럼 동작한다. value가 숫자형이 아니면 null이 result 할당된다

### Destructuring 을 위한 일반 class 선언

```kotlin
// 일반 클래스는 componentN 메소드를 구현 해야 한다.
class Person(val name: String, val age: Int) {
    operator fun component1(): String = name
    operator fun component2(): Int = age
}

val person = Person("Adam", 100)
val (name, age) = person

assertEquals("김찬정", name)
assertEquals(20, age)
```

### Destructuring 을 위한 data class 선언
```kotlin
// 데이터 클래스는 기본으로 componentN 제공한다.
data class NewPerson(val age: Int, val name: String) 

val newPerson = NewPerson(20, "김찬정")
// 프로퍼티 이름이 다른경우 선언 순서대로 리턴한다
val (newName, newAge) = newPerson

assertEquals(20, newName) // age가 먼저 선언 되었으므로 사실상 age값이 된다.
assertEquals("김찬", newAge)
```

### Destructuring 배열
```kotlin
// 배열
val coordinates = arrayOf(1, 2, 3)
val (x, y, z) = coordinates
```







