---
layout: post
title: "[kotlin] 코틀린 기본 분법 간략 요약"
description: 코틀린 기본 분법 간략 요약
author: kimchanjung
date: 2020-04-29 00:00:00 +0900
categories: kotlin
published: true
---
## 코틀린 기본 분법 간략 요약

kotlin 기본 문법을 간략하게 파악 해보기 위한 요약 정리 입니다. kotlin 문법의 모든 내용을 다루지는 않지만 간략하게 훝어 보는 용도로 포스팅합니다.

### 코틀린 변수 선언

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

### FOR

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

### WHILE

```kotlin
 val item = Array(5) { v -> v + 1 }
 var index = 0

 while (index < item.size) {
     println(item[index])
     index++
 }
```

### 함수 선언
```kotlin
// 기본적인 함수 선언 방식
// fun basicFunc(name: String): Int <= Int는 리턴 타입이다
fun basicFunc(name: String): Int {
    return name.toInt()
}

// 한줄에 선언 하는 방식
// name: String = "김찬정" <= 매개변수에 기본 값을 선언, 함수 호출시 매개변수가 주어지지 않으면
// 기본값으로 선언된다.
// 한줄 선언 식에서는 리턴타입 생략 가능하다.
fun simpleFunc(name: String = "김찬정", age: Int = 10) = if (age > 10) name else "어린이"
```

### 함수호출 시 가변인자

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
확장함수는 특정클래스로 부터 상속받지 않고 해당클래스의 기능을 확장 한다.

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

### 중위함수(infix)
#### 중위함수 선언 조건
- 클래스의 멤버 함수이거 나 확장 함수이어야 한다. 
- 매개변수가 한 개여야 한다.
- infix 키워드로 함수가 정의되어야 한다.

```kotlin
// 중의 함수 선언은 infix 키워드를 붙여 선언한다.
infix fun Int.multiply(value: Int): Int {
    return this * value
}

println(9 multiply 1) // 10 출력
println(9.multiply(1)) // 10 출력
println(3 + 2 multiply 2) // 10 출력, + 연산자 우선순위가 더 높다 

```

### 클래스 선언

#### 간단한 클래스 선언
kotlin 클래스의 기본생성자는 클래스명 바로 옆 "()" 구문이다.  
기본 생성자의 매개변수 선언을 val name: String 하면 멤버 프로퍼티(변수)선언과 기본생성자 매개변수 선언을 동시에 한 것이다.

#### kotlin 버전
```kotlin
// 멤버 프로퍼티(변수)와 생성자를 동시에 선언
class SimpleClass(val name: String, var address: String, var age: Int = 41)
```
다음의 자바 버전의 클래스 선언과 동일하다.

#### java 버전
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

### 클래스 선언과 생성자
#### 생성자 매개변수 선언 방식 차이
```kotlin
// var name: String <= var/val 를 붙이면 
// 생성자 매개변수 선언과 동시에 멤버 프로퍼티(변수)를 선언한 것과 같다
class BasicClass(var name: String, var address: String) {
    // var name:String <= 한 것 과 같다
    ...
}

// name: String <= var/val 를 붙이지 않으면 생성자 매개변수만 선언 한 것 이다.
// 그래서 멤버 프로퍼티(변수)를 꼭 선언 해야한다.
class BasicClass(name: String, address: String) {
    var name:String
    var address:String

    ...
}
```

#### 기본생성자의 초기화 init 블럭
class BasicClass(var name: String, var address: String) 처럼 프로퍼티 선언과 생성자 매개변수 동시 선언 하지 않으면 아래 예제 처럼 멤버 프로퍼티 선언과 함께 init블럭을 통하여 멤버프로퍼티의 초기화도 해주어야 한다.
```kotlin
class BasicClass(name: String, address: String, age: Int) {
    // 프로퍼티선언과 동시에 생성자 파라메터로 값 설정이 가능하며 초기화 구문 까지 포함 된 것이다.
    var name: String = name 
    var address: String
    var age: Int

    /**
     * 초기화 블럭
     * 디폴트 생성자의 함수의 초기화 블럭이라고 보면 된다. java로 보면 아래와 같다
     *
     * public class BasicClass {
     *     public BasicClass(String name, String address, Intger age) {
     *          this.name = name;
     *          this.address = address;
     *          this.age = age;
     *     }
     * }
     */
    init {
        /**
         * name 프로퍼티는 
         * var name: String = name <= 선언과 동시에 초기화 구문 까지 함께 기술 하였기 때문에
         * init 블럭에서 초기화 구분이 제외 되었다.
         */
        // this.name = name 
        this.address = address
        this.age = age
    }
}
```
### 클래스와 접근제한자

|접근 제한자| 클래스 멤버일 때 |최상위 수준으로 선언되었을 때|
|--|--|--|
|public(기본값)|어디서든 사용 가능|어디서든 사용 가능|
|internal|같은 모듈에서만 사용 가능|같은 모듈에서만 사용 가능|
|protected|서브 클래스에서만 사용 가능|해당 없음|
|private|클래스 내부에서만 사용 가능|코틀린 파일 내부에서만 사용 가능|

### 클래스의 프로퍼티와 getter, setter

#### 코들린의 프로퍼티란 ?
코틀린에서 프로퍼티란 클래스의 멤버 변수를 말하는데 코틀린에서 멤버 변수는 단순히 **var name:String** 으로 선언 되어 **변수만 선언된 것 같지만** setter/getter를 따로 선언 하지 않아도 **기본적으로 생성**되며 **변수/setter/getter를 프로퍼티**라고 부른다.  
하지만 person.name 처럼 사용하면 person.getName() 처럼 getter를 사용하지 않는 것 처럼 보이지만 사실은 person.name 값을 가져오면 내부적으로 getter를 사용 한 것이다. 

```kotlin
class BasicClass(name: String, address: String, age: Int) {
   
    var name: String 
    var address: String
    // 기본적으로 getter가 제공되지만 아래 구문은 기본 getter를 오버라이드 한 것이다.
        get() {
            return "$field 특별시"
        }
     // 기본적으로 setter 제공되지만 아래 구문은 기본 setter 오버라이드 한 것이다.    
        set(address: String) {
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
// basicClass.getAddress(), basicClass.setAddress("서울")
// 처럼 사용해야 될 것 같지 만 아래처럼 사용 하며, 실제로 getter/setter가 동작 한다.
println(basicClass.address) // 서울 특별시 로 출력 됨
basicClass.address = "서울" // 서울 특별시 로 변경됨

```
#### 코들린의 public 프로퍼티의 변수는 public인가?

코틀린의 접근제한자는 기본적으로 멤버 프로퍼티(변수+getter+setter)에 대한접근 제한 자이다. public var name:String 은 name라는 변수는 private로 직접 접근이 불가능하며 기본으로 제공되는 getter/setter에 의해서 접근이 되는 것이고 getter/setter가 제공됨으로써 name프로퍼티는 public인 것 이다. 아래 예제를 보자

```kotlin
// 아래 처럼 선언하면 
class BasicClass(name:String) {
    var name:String // public 생략됨
    private var age: Int // 프로퍼티를 private로 선언하면
}

// JAVA 코드로 변환 하면 실제로는 아래와 같다

public class BasicClass {
    private String name;
    private Integer age;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    /**
    private var age: Int <= 프로퍼티를 private로 선언하면 아래의
    getter/setter가 제공 되지 않아 외부에서 접근 자체가 안되는 것이다.
    public Integer getAge() {
        return this.age;
    }

    public voud setName(Integer age) {
        this.age = age;
    }
    */
    
}
```

### 생성자 오버로딩(여러개의 생성자)
#### lazyinit 이란?
멤버프로퍼티의 값 초기화를 지연하는 키워드 
- var(mutable) 프로퍼티만 사용 가능
- non-null 프로퍼티만 사용 가능
- 커스텀 getter/setter가 없는 프로퍼티만 사용 가능
- primitive type 프로퍼티는 사용 불가능
- 클래스 생성자에서 사용 불가능
- 로컬 변수로 사용 불가능

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


### Init 블럭 없는 생성자 오버로딩
클래스 선언시 클래스명 옆에 기본생성자를 선언하지 않고 constructor를 선언한다. 그렇게 되면 init 블럭도 필요없어진다. 
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

### 정적 팩토리 메소드
- 기본생성자를 private로 접근제한하여 외부에 제공하지 않는다.
- companion object 사용하여 static 메소드를 생성하여 객체를 생성하는 정적 팩토리를 제공한다.  

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
JAVA 코드로 보자면 아래와 같다. 

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
위 처럼 코들린 클래스의 정적팩토리 메소드를 선언하면 **자바에서 코틀린 객체의 스태틱멤버를 호출 할때 PrivateDefaultConstructor.companion.address** 로 접근 해야 하기 때문에 아래 예제 처럼 선언 해야 **PrivateDefaultConstructorForJava.address** 로 접근 가능하다.  

```kotlin
// static field에는 const 또는 @JvmField 붙인디
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
