---
layout: post
title: "[Spring Security] 커스텀 필터를 이용한 Google 인증 구현 - 스프링시큐리티 설정(2)"
description: "spring security custom filter google auth structure configuration"
author: kimchanjung
date: 2020-07-02 23:00:00 +0900
categories: spring
published: true
---

# Spring Security 커스텀 필터를 이용한 Google 인증 구현 - 스프링시큐리티 설정(2)
> 본 포스팅은 스프링시큐리티의 전반적인 사용방법을 설명하는 포스팅은 아닙니다. 기본적인 동작구조와 Google 인증 같은 외부 인증을 도입할 때 필요한 커스텀인증필터를 작성하고 적용하는 방법을 알아봅니다.

## 버전정보
- Spring Boot v2.1.2
- Spring Security v 5.1.3

## Spring Security 설정
스프링부트를 사용한다면 application.yml에서 설정도 가능하지만 몇가지 설정만 제공하고 모든 설정을 할 수 없으므로 `Configuration Class에서 하는 설정이 기본`이라고 생각하시면 됩니다.

## Configuration Class 작성
스프링시큐리티 Configuration Class를 작성하기 위해서는 WebSecurityConfigurerAdapter를 상속하여 클래스를 생성하고 @EnableWebSecurity 애노테이션을 추가합니다. (@Configuration 애노테이션 대신)
```java
@EnableWebSecurity
public class BrmsWebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    /**
     * 스프링시큐리티 앞단 설정들을 할 수 있다.
     */
    @Override
    public void configure(WebSecurity web) throws Exception {
        // resources 모든 접근을 허용하는 설정을 해버리면
        // HttpSecurity 설정한 ADIM권한을 가진 사용자만 resources 접근가능한 설정을 무시해버린다.
        web.ignoring()
                .antMatchers("/resources/**");
    }

    /**
     * 스프링시큐리티의 설정을 할 수 있다.
     */
    @Override
    public void configure(HttpSecurity http) throws Exception {
        // WebSecurity에 접근 허용 설정을 해버리면 이 설정이 적용되지 않는다.
        http.authorizeRequests()
                .antMatchers("/resources/**").hasRole("ADIM") // no effect
                .anyRequest().authenticated();
    }
}    
```
[코드1] 스프링시큐리티 설정 
### WebSecurity 설정
> 스프링시큐리티의 각종 설정은 HttpSecurity로 한다  WebSecurity 스프링시큐리티 앞단의 설정들을 하는 객체이므로 HttpSecurity 설정한 스프링시큐리티 설정이 오버라이드 되는 설정이 있는 경우도 있다.  

`기억해야될 부분은 스프링시큐리티의 설정은 HttpSecurity 하는 것 입니다.` 
WebSecurity 때문에 햇갈림이 발생할 수 있는 부분이기도 합니다. WebSecurity는 잠시 잊으셔도 좋습니다.

### HttpSecurity 설정
> 스프링시큐리티의 각종 설정은 HttpSecurity로 대부분 하게 됩니다. 
> - 리소스(URL) 접근 권한 설정 
> - 인증 전체 흐름에 필요한 Login, Logout 페이지 인증완료 후 페이지 인증 실패 시 이동페이지 등등 설정
> - 인증 로직을 커스텀하기위한 커스텀 필터 설정
> - 기타 csrf, 강제 https 호출 등등 거의 모든 스프링시큐리티의 설정   

`HttpSecurity는 스프링시큐리티의 거의 대부분설정을 담당`하는 객체 입니다.

## 간략한 스프링시큐리티 설정들
세세한 설정 정보는 매우 방대 하기 때문에 공식 문서를 참고하기 바랍니다. 이 내용에서는 기본적인 몇가지 설정 방법들을 설명합니다.

### 리소스(URL)의 권한한 설정
특정 리소스의 접근 허용 또는 특정 권한을 가진 사용자만 접근을 가능하게 할 수 있습니다. 
```java
@EnableWebSecurity
public class BrmsWebSecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/login**", "/web-resources/**", "/actuator/**").permitAll()
                .antMatchers("/admin/**").hasAnyRole("ADMIN")
                .antMatchers("/order/**").hasAnyRole("USER")
                .anyRequest().authenticated();
    }
}    
```
[코드2] 리소스의 인증 및 권한 설정

#### antMatchers
```java
 antMatchers("/login&#42;&#42;", "/web-resources/&#42;&#42;", "/actuator/&#42;&#42;")
```
특정 리소스에 대해서 권한을 설정합니다.

#### permitAll
```java
antMatchers("/login&#42;&#42;", "/web-resources/&#42;&#42;", "/actuator/&#42;&#42;").permitAll() 
```
antMatchers 설정한 리소스의 접근을 인증절차 없이 허용한다는 의미 입니다.

#### hasAnyRole
```java
antMatchers("/admin/&#42;&#42;").hasAnyRole("ADMIN")
```
리소스 admin으로 시작하는 모든 URL은 `인증후 ADMIN 레벨의 권한을 가진 사용자만 접근을 허용`한다는 의미입니다.

#### anyRequest
```java
anyRequest().authenticated()
```
모든 리소스를 의미하며 접근허용 리소스 및 인증후 특정 레벨의 권한을 가진 사용자만 접근가능한 리소스를 설정하고 `그외 나머지 리소스들은 무조건 인증을 완료해야 접근이 가능`하다는 의미입니다.

### 로그인처리 설정
가장 일반 적인 로그인 방식인 `로그인 FORM 페이지를 이용하여 로그인하는 방식을 사용`하려고 할때 여러가지 설정을 할 수 있습니다. 중요한 사실은 `커스텀 필터를 적용 할때와 여러가지 설정이 중복되거나 서로 상관없는 설정이 겹치게 되어 햇갈는 부분`이 있습니다.
```java
@EnableWebSecurity
public class BrmsWebSecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .loginPage("/login-page")
                .loginProcessingUrl("/login-process")
                .defaultSuccessUrl("/main")
                .successHandler(new CustomAuthenticationSuccessHandler("/main"))
                .failureUrl("login-fail")
                .failureHandler(new CustomAuthenticationFailureHandler("/login-fail"))
                
    }
} 
```
[코드3] 로그인 폼 사용 및 설정
#### formLogin
> 로그인 페이지와 기타 로그인 처리 및 성공 실패 처리를 사용하겠다는 의미 입니다.

```java
http.formLogin();  
```
일반적인 로그인 방식 즉 로그인 폼 페이지와 로그인 처리 성공 실패 등을 사용하겠다는 의미입니다. `http.formLogin() 를 호출하지 않으면`
완전히 로그인처리 `커스텀필터를 만들고 설정하지 않는 이상 로그인 페이지 및 기타처리를 할 수 가 없습니다`. 커스텀 필터를 만들면 사실상 필요 없는 경우도 있습니다. 

#### loginPage
> 사용자가 따로 만든 로그인 페이지를 사용하려고 할때 설정합니다.

```java
loginPage("/login-page") 
```
따로 설정하지 않으면 디폴트 URL이 **"/login"**이기 때문에 **"/login"**로 호출하면 스프링이 제공하는 기본 로그인페이지가 호출됩니다.

#### loginProcessingUrl
> 로그인 즉 인증 처리를 하는 **URL**을 설정합니다. **"/login-process"** 가 호출되면 인증처리를 수행하는 필터가 호출됩니다.

```java
loginProcessingUrl("/login-process") 
```
로그인 FORM에서 아이디와 비번을 입력하고 확인을 클릭하면 **"/login-process"** 를 호출 하게 되었들 때 `인증처리하는 필터가 호출`되어 아이디 비번을 받아와 인증처리가 수행되게 됩니다. 즉 `UsernamePasswordAuthenticationFilter가 실행` 되게 되는 것입니다.

#### defaultSuccessUrl
> 정상적으로 인증성공 했을 경우 이동하는 페이지를 설정합니다.

```java
defaultSuccessUrl("/main")
```
설정하지 않는경우 디폴트값은 "/" 입니다.

#### successHandler
> 정상적인증 성공 후 별도의 처리가 필요한경우 커스텀 핸들러를 생성하여 등록할 수 있습니다.

```java
successHandler(new CustomAuthenticationSuccessHandler("/main"))
```
커스텀 핸들러를 생성하여 등록하면 인증성공 후 사용자가 추가한 로직을 수행하고 성공 페이지로 이동합니다.

#### failureUrl
> 인증이 실패 했을 경우 이동하는 페이지를 설정합니다. 

```java
failureUrl("/login-fail")
```
#### failureHandler
> 인증 실패 후 별도의 처리가 필요한경우 커스텀 핸들러를 생성하여 등록할 수 있습니다.

```java
successHandler(new CustomAuthenticationFailureHandler("/login-fail"))
```
커스텀 핸들러를 생성하여 등록하면 인증실패 후 사용자가 추가한 로직을 수행하고 실패 페이지로 이동합니다.

## 커스텀 필터 등록
스프링시큐리티는 `각각역할에 맞는 필터들이 체인형태로 구성되서 순서에 맞게 실행`되는 구조로 동작합니다. 사용자는 특정 기능의 필터를 생성하여 등록할 수 있습니다. `인증을 처리하는 기본필터 UsernamePasswordAuthenticationFilter` 대신 `별도의 인증 로직을 가진 필터를 생성하고 사용하고 싶을 때 아래와 같이 필터를 등록`하고 사용합니다.
```java
@EnableWebSecurity
public class BrmsWebSecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.addFilterBefore(new CustomAuthenticationProcessingFilter("/login-process"), 
                UsernamePasswordAuthenticationFilter.class);         
    }
} 
```
[코드4] 커스텀 필터의 추가 

#### addFilterBefore  
> 지정된 필터 앞에 `커스텀 필터를 추가 (UsernamePasswordAuthenticationFilter` 보다 먼저 실행된다)

#### addFilterAfter 
> 지정된 필터 뒤에 커스텀 필터를 추가 (UsernamePasswordAuthenticationFilter 다음에 실행된다.)

#### addFilterAt  
> 지정된 필터의 순서에 커스텀 필터가 추가된다

마치 지정된 필터 대신에 커스텀필터를 추가하는 것 처럼 메소드가 동작할 것 같지만 `실제로는 오버라이드 되지는 않습니다`.
설명에는 `지정된 필터의 순서와 같은 자리에 커스텀필터를 삽입`한다고 되어 있고 `오버라이드 되지 않는다고 설명`되어 있습니다.
하지만 직접테스트 결과 `지정된 필터 보다 커스텀 필터가 먼저 실행`되었습니다.   
결론 적으로 어떤 메소드로 `커스텀 필터를 추가하더라도 기존 필터가 오버라이드 되는 메소드는 없습니다`. 다만 `커스텀 필터가 실행 되고 인증이 완료` 되었기 때문에 `UsernamePasswordAuthenticationFilter 수행`되면서 `인증완료된 상태이면 인증 로직이 수행되지 않고 자연스럽게 통과` 하기 때문에 마치 오버라이드된 것 처럼 동작하는 것으로 착각 할 수 있습니다 (직접 테스트 해봄)



### 커스텀필터에 설정을 추가
커스텀 필터에 인증 매니저 및 성공 실패 핸들러를 추가적으로 등록 및 설정을 추가 할 수 있습니다.
```java
@EnableWebSecurity
public class BrmsWebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    public CustomAuthenticationProcessingFilter customAuthenticationProcessingFilter() {
        CustomAuthenticationProcessingFilter filter = new CustomAuthenticationProcessingFilter("/login-process");
        filter.setAuthenticationManager(new CustomAuthenticationManager());
        filter.setAuthenticationFailureHandler(new CustomAuthenticationFailureHandler("/login"));
        filter.setAuthenticationSuccessHandler(new SimpleUrlAuthenticationSuccessHandler("/"));
        return filter;
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.addFilterBefore(customAuthenticationProcessingFilter(), 
                UsernamePasswordAuthenticationFilter.class);         
    }
} 
```
[코드5] 커스컴 필터의 상세 설정

#### CustomAuthenticationProcessingFilter
> AbstractAuthenticationProcessingFilter 상속하여 구현하며 인증처리를 담당합니다.

```java
http.addFilterBefore(customAuthenticationProcessingFilter(), 
        UsernamePasswordAuthenticationFilter.class);  
```
스프링시큐리티의 기본 인증 처리 담당 필터인 UsernamePasswordAuthenticationFilter 앞에 커스텀 필터를 추가합니다.  
`UsernamePasswordAuthenticationFilter필터는 Override되지 않고` CustomAuthenticationProcessingFilter 인증 처리가
되면 `자연스럽게 로직을 통과`하는 구조입니다.  
인증로직이 포함된 `인증매니저 인증성공/실패 처리 핸들러` 등등 기타 추가 설정 이나 기능을 대체 하는 의존성을 주입 할 수 있습니다.

#### setAuthenticationManager
> AuthenticationManager를 상속받아 생성하며 실제 인증로직이 포함된 커스텀 인증 매니저를 추가합니다.

```java
filter.setAuthenticationManager(new CustomAuthenticationManager());
```  

#### setAuthenticationFailureHandler
> 인증실패 핸들러를 따로 등록 할 수 있습니다. 따로 등록하지 않으면 기본 핸들러가 동작합니다.

```java
filter.setAuthenticationFailureHandler(new CustomAuthenticationFailureHandler("/login"));
```


#### setAuthenticationFailureHandler
> 인증실패 핸들러를 따로 등록 할 수 있습니다. 따로 등록하지 않으면 기본 핸들러가 동작합니다.

```java
filter.setAuthenticationFailureHandler(new CustomAuthenticationFailureHandler("/login"));
```


### 마무리
스프링시큐리티의 기본적인 설정 방법과 커스텀 필터를 등록하는 방법을 알아 보았습니다. 설정의 세부 내용이 방대하기 때문에 공식 문서를 참고해야합니다.
본 글에서는 대략적인 설정 내용을 알아 보았습니다. 다음 포스팅에는 `커스텀 필터를 생성해서 구글인증(별도의 인증로직을 구현)같이 사용자가 별도의 
인증로직을 구현`하는 커스텀 필터의 세부 내용을 알아보도록 하겠습니다.