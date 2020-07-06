---
layout: post
title: "[Spring Security] 커스텀 필터를 이용한 인증 구현 - 스프링시큐리티 동작구조의 이해(1)"
description: "스프링시큐리티의 기본적인 동작구조와 별도의 인증을 도입할 때 필요한 커스텀인증필터를 작성하고 적용하는 방법을 알아봅니다"
author: kimchanjung
date: 2020-07-01 01:00:00 +0900
categories: spring
iimages: ["/post-img/spring-security/spring-security-flow-diagram.png", "/post-img/spring-security/spring-security-flow-diagram.png"]
published: true
---

# Spring Security 커스텀 필터를 이용한 인증 구현 - 스프링시큐리티 동작구조의 이해(1)
> 본 포스팅은 스프링시큐리티의 전반적인 사용방법을 설명하는 포스팅은 아닙니다. 기본적인 동작구조와 별도의 인증을 도입할 때 필요한 커스텀인증필터를 작성하고 적용하는 방법을 알아봅니다.

## 버전정보
- Spring Boot v2.1.2
- Spring Security v 5.1.3

## 러닝커브가 높은 스프링시큐리티
본인도 처음에 **실무 프로젝트에 스프링시큐리티를 이용하여 인증을 구현**하면서 느끼는 것 이지만 인증자체가 비즈니스로직처럼 **매번 개발하는 부분이 아니기 때문에 익숙하지 않고** 팀내에서 이미 개발되어 있는 경우에는 **신규 프로젝트를 할 때 이외엔 볼일도 별로 없는 부분**이기도 합니다.  
> 스프링시큐리티는 **인증과 권한등의 많은 기능을 편리하게 적용**할 수 있도록 도와 주는 라이브러리이지만 **사용법은 결코 만만하지 않다는 느낌을 받습니다.** 스프링과 JAVA공식 문서들이 그렇지만 **방대한 내용과 설명이 많아** 학습하는 것도 쉽지 않습니다. (요즘 React나 Vue같은 프론트엔드 공식문서는 너무도 잘되어 있는 최근 추세에 비해서)  

## 가장 먼저 학습 해야하는 부분
그래서 저는 일단 공식 문서를 처음부터 정독하거나, 튜토리얼을 보는 것도 좋지만 `가장 먼저 스프링시큐리티의 동작 구조를 먼저 이해`하는 것이 나중에 학습 + 개발의 리소스를 줄여주는 방법이라고 생각합니다. 
> 스프링시큐리티의 동작 구조를 이해하면 **설정과 추가적인 구현 및 변경이 어느 부분에 적용되어야 되는지 파악이 빨라** 집니다.

## 스프링시큐리티의 동작 구조
> 스프링시큐리티는 각각의 **역할에 맞는 작업을 처리하는 여러개의 필터들이 체인형태로 구성**되어 순서에 따라 순차적으로 수행됩니다.   

그 중 **UsernamePasswordAuthenticationFilter**가 `인증처리를 담당`하고 있습니다.  
#### Spring Security Flow Communication Diagram
![spring-security-flow-diagram](/post-img/spring-security/spring-security-flow-diagram.png)
[그림1] 스프링시큐리티 동작구조

#### 1. 로그인정보를 담아 서버에 인증을 요청한다.
```java
 1: doFilter(request: HttpServletRequest, response: HttpServletResponse)
```  
브라우저의 로그인화면에서 아이디와 비번을 입력하고 확인을 누르면 서버에 로그인 인증 요청을 하게 되고 
스프링시큐리티에 `Chain형태로 구성된 Filter들의 doFilter메소드들이 순서에 따라 호출`되어 각각의 역할로직들이 수행되게 됩니다.

#### 2. 인증 처리 담당하는 UsernamePasswordAuthenticationFilter 실행된다.
```java
2: attemptAuthentication(request: HttpServletRequest, response: HttpServletResponse):Authentication  
```  
- **AbstractAuthenticationProcessingFilter 추상 클래스의 추상메소드 attemptAuthentication**를 호출하여 요청을 처리하게 됩니다. attemptAuthentication추상메소드의 구현은 상속한 UsernamePasswordAuthenticationFilter에 구현 되어 있습니다. 
- 추후 `외부인증을 위한 작업을 할때 UsernamePasswordAuthenticationFilter기능을 대신하는 커스텀 필터를 추가`하게 됩니다. 
- 인증 성공 실패에 따라 AuthenticationSuccessHandler, AuthenticationFailureHandler 를 최종적으로 호출하게 됩니다.
- 인증이 성공했다면 리턴값 UsernamePasswordAuthenticationToken를 세션에 저장합니다


#### 3. AuthenticationManager가 적절한 AuthenticationProvider를 찾는다.
```java
 3: authenticate(authRequest):Authentication  
``` 

- AuthenticationManager는 인터페이스이며 구현체는 ProviderManager입니다. `ProviderManager`
는 실제 `인증을 처리하는 로직이 포함된 AuthenticationProvider 인터페이스의 구현체`들 중에 설정된 인증 처리방식의 구현체를 찾아 실행합니다.

#### 4. 실제 인증처리하는 AuthenticationProvider의 인증처리 메소드를 호출한다.
```java
4: authenticate(authRequest):Authentication   
```   
일반적으로 클라이언트에서 아이디 비번을 받아 인증하는 방식을 사용한다면 AuthenticationProvider 인터페이스의 구현체 AbstractUserDetailsAuthenticationProvider 추상클래스를 호출하게 되고 실제 상속한 클래스의 DaoAuthenticationProvider에서 인증 처리를 하게 됩니다. 

#### 5. 인증제공자는 UserDetailsService를 호출하여 사용자를 가져온다.
```java
5: loadUserByUsername(username:String):UserDetails  
```  
개발자는 UserDetailsService 인터페이스를 구현해야합니다. UserDetailsService의 구현체에는 일반적으로 회원정보가 DB에 있다고 한다면 사용자의 이름(ID)로 DB를 조회하여 비밀번호가 일치하는지 확인하여 인증을 처리합니다. 인증이 완료되면 UsernamePasswordAuthenticationToken에 회원정보를 담아 리턴하게 됩니다.

### 마무리
인증을 구현하기 앞서 스프링시큐리티의 대략적인 동작구조를 알아보았습니다. 다음 포스팅에서는 본격적으로 어떤 부분을 커스텀해야 하는지에 대해 알아 보도록 하겠습니다.
