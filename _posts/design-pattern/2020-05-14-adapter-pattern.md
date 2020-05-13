---
layout: post
title: "어탭터 패턴 - Adapter Pattern [디자인패턴]"
description: 어탭터 패턴 - Adapter Pattern [디자인패턴][디자인패턴]
author: kimchanjung
date: 2020-05-14 00:00:00 +0900
categories: design pattern
published: true
---

# 어탭터 패턴 - Adapter Pattern

## 어탭터 패턴이란
> 사용자는 일관성있는 인터페이스를 사용하여 구현체의 변경으로 부터 자유롭고   
> 인터페이스의 구현체는 각각 세부로직은 다르지만 인터페이스에 맞게 구현하여  
> 사용자는 약간식 세부 구현이 다른 클래스를 일관성있게 사용하는 패턴이다.

## 이해를 돕기위한 설명 
### 어댑터 패턴 미적용 시
- 로그인기능이 있다 구글인증을 사용해 로그인 하다가 페이스북 인증이 추가 되었다.
- 구글과 페이스북은 인증 절차가 상이하다.
- 구글인증이 필요하면 구글객체를 생성하고 인증한다.
- 페이스북인증이 필요하면 페이스북객체를 생성하고 인증한다.

```bash
로그인(타입)
    if(타입 == 구글)
        구글 구글 = new 구글()
        구글.로그인페이지()
        구글.인증()
        구글.리다이렉트()
    else 
        페이스북 페이스북 = new 페이스북()
        페이스북.로그인페이지()
        페이스북.로그인()
    ...
```

### 어댑터 패턴 적용
- 나는 각각 필요한 인증업체의 인증 객체를 따로 사용하지 않고 만능 리모컨 같은
하나의 인증서비스를 이용하고 싶다.
- 어댑터 패턴을 이용하면 서로다른 인증업체의 객체를 추상화 하여 사용자에게
- 일반화된 인터페이스를 제공함으로써 인증업체의 종류에 따라 별개의 객체를 할 필요가
없어진다.
- 인증업체가 추가 되거나 세부 로직이 변경되어도 클라이언트의 코드는 수정할 필요가 없다.

```bash
로그인(로그인서비스)
    로그인서비스.로그인페이지()
    로그인서비스.인증()
    로그인서비스.리다이렉트()
```

## 장단점
### 클래스 방식 - 상속이용 
장점 - 어댑터(Adapter)를 전체를 다시 구현할 필요가 없다.(빠르다)  
단점 - 상속(Generalzation)을 활용하기때문에 유연하지 못하다.  
### 오브젝트 방식속 - 인터페이스 이용 
장점 - 구성(Composition)을 사용하기 때문에 더 뛰어나다.(유연하다)  
단점 - 어댑터(Adapter)클래스의 대부분의 코드를 구현해야하기때문에 효율적이지 못하다.

## 클래스다이어 그램
![class-diagram](/post-img/design-pattern/adapter-pattern-class-diagram.png)

## 예제코드
### 로그인서비스를 추상화
```kotlin
/**
 * 어댑터 인터페이스를 선언한다
 */
interface LoginAdapter {
    fun goLoginPage(): LoginAdapter
    fun requestLogin(id: String, pw: String): LoginAdapter
    fun redirect(url: String): String
}

/**
 * 각각 어탭터 인터페이스를 구현한다
 * 기존 로그인 서비스를 사용하는 구현체
 */
class GoogleLoginAdapterImpl : LoginAdapter {
    private val googleLoginService = GoogleLoginService()

    override fun goLoginPage(): LoginAdapter {
        googleLoginService.goLoginPage()
        return this
    }

    override fun requestLogin(id: String, pw: String): LoginAdapter {
        googleLoginService.requestLogin(id, pw)
        return this
    }

    override fun redirect(url: String) = googleLoginService.redirect(url)
}

class FaceBookLoginAdapterImpl : LoginAdapter {
    private val faceBookLoginService = FaceBookLoginService()

    override fun goLoginPage(): LoginAdapter {
        faceBookLoginService.goLoginPage()
        return this
    }

    override fun requestLogin(id: String, pw: String): LoginAdapter {
        faceBookLoginService.requestLogin(id, pw)
        return this
    }

    override fun redirect(url: String): String {
        // 페이스북 로그인은 redirect하지 않기 때문에 빈 메소드를 정의한다
        return url
    }
}
```
> 구글, 페이스북 인증을 추상화한 인터페이스를 클라이언트에게 제공하고 개별 세부 구현체를 생성한다. 

### 추상화된 로그인 서비스를 사용하는 클라이언트
```kotlin

class LoginService(private val loginAdapter: LoginAdapter) {
    fun login(id: String, pw: String, redirectUrl: String) = loginAdapter
            .goLoginPage()
            .requestLogin(id, pw)
            .redirect(redirectUrl)
}

val clientService = LoginService(FaceBookLoginAdapterImpl())
val clientService2 = LoginService(GoogleLoginAdapterImpl())

clientService.login("id", "pw,", "www.home.com")
clientService.login("id", "pw,", "new.home.com")
```
> 클라이언트는 어떤 로그인서비스를 사용할지 스스로 판단단하여 필요한 로직을 또 추가 구현 할 필요없이 로그인 어댑터 인터페이스만 사용하면 된다.
> 인증업체가 추가되어도 클라이언트코드는 변경이 없다.