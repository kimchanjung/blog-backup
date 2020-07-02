---
layout: post
title: "[Spring Security] 커스텀 필터를 이용한 인증 구현 - 커스텀 필터의 구현(3)"
description: "spring security custom filter auth structure configuration"
author: kimchanjung
date: 2020-07-02 19:00:00 +0900
categories: spring
published: true
---

# Spring Security 커스텀 필터를 이용한 인증 구현 - 커스텀 필터의 구현(3)
> 본 포스팅은 스프링시큐리티의 전반적인 사용방법을 설명하는 포스팅은 아닙니다. 기본적인 동작구조와 별도의 인증을 도입할 때 필요한 커스텀인증필터를 작성하고 적용하는 방법을 알아봅니다.

## 버전정보
- Spring Boot v2.1.2
- Spring Security v 5.1.3

## 커스텀 인증을 위한 클래스들을 생성
스프링시큐리티가 제공하는 인증필터와 인증매니저 대신 별도의 인증처리를 담당할 커스텀 클래스들을 생성합니다.

### 커스텀 필터 CustomAuthenticationProcessingFilter 생성
전체적인 인증 처리를 담당합니다. 

```java
public class CustomAuthenticationProcessingFilter extends AbstractAuthenticationProcessingFilter {

    public CustomAuthenticationProcessingFilter(String defaultFilterProcessesUrl) {
        super(defaultFilterProcessesUrl);
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        String username = request.getParameter("username");
        String password = request.getParameter("password");

        return getAuthenticationManager()
                .authenticate(new UsernamePasswordAuthenticationToken(username, password)));
    }
}
```
[코드1] 커스텀 인증 필터 생성 
> - defaultFilterProcessesUrl에 설정한 URL이 호출되면 수행됩니다. 예) "/login-process"
> - username, password를 받아와서 UsernamePasswordAuthenticationToken에 담아 실제 인증처리를 하는 인증매니저를 호출합니다.
> - `username, password를 가져오는 로직의 예외처리`는 필수 입니다. 핵심로직만 설명하기 위해 간략한 코드로 설명합니다.


## 커스텀 인증매니저 CustomAuthenticationManager를 생성
`커스텀 필터는 전체적인 인증처리를 담당`하는 역할을 하고 `인증매니저는 실제 인증처리를 하는 로직을 구현`하는 클래스입니다.

### CustomAuthenticationManager
```java
public class CustomAuthenticationManager implements AuthenticationManager {

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        UserDetails userDetails = userDetailsService.loadUserByUsername(authentication.getPrincipal());
        // 각종 처리를 구현
        // 비번이 일치하는지
        // 아이디로 회원을 조회 했을 때 존재하는 회원인지
        // 기타 등등과 적절한 예외 처리 
        return new UsernamePasswordAuthenticationToken(userDetails.getUsername()
                , userDetails.getPassword()
                , userDetails.getAuthorities()))
    }
}
```
[코드2] 커스텀 인증 매니저 생성
> - AuthenticationManager를 상속한 커스텀 인증매니저는 로그인 폼으로 부터 받아온 아이디로 사용자를 가져와 인증 처리를 한다. 
> - `비번이 일치하는 지, 존재하는 사용자인지 등등의 로직을 처리`하고 적절한 예외 처리도 한다.
> - `코드의 핵심을 설명`하기 위해 `상세한 로직 처리와 예외 처리는 생략`하였습니다.

## 사용자정보를 가져오는 인터페이스를 구현 
UserDetailsService는 스프링시큐리티가 제공하는 인터페이스입니다. 인터페이스에 맞게 사용자가 구현해야 하며 `사용자의 정보를 가져오는 로직을 구현`합니다.
사용자의 정보를 가져오는 로직은 `DB가 될 수도 있고 별도의 인증서버일 수도 있고 사용자의 상황에 맞게 구현`하면 됩니다.

### UserDetailsService 구현
#### 유저정보를 담아낼 UserDetails인터페이스를 구현
스프링시큐리에서 제공하는 UserDetails를 구현하여 유저정보를 담는 클래스를 생성합니다.
```java
public class CustomUserDetails implements UserDetails {
    private List<GrantedAuthority> authorities;
    private String password;
    ...

    public CustomUserDetails(User user) {
        this.authorities = user.getAuthorities();
        this.password = user.getPassword();
        ....
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    ...
}    
```
[코드3] 유저 정보를 담는 클래스

#### UserDetailsService를 구현
유저정보를 아이디로 찾아와서 UserDetails구현한 CustomUserDetails클래스에 담아서 리턴하는 로직을 구현합니다.

```java
@Service
public class UserDetailServiceImpl implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String userId) throws BrmsBadCredentialsException {
        return CustomUserDetails(userRepository.findByUserId(userId));
    }
}
```
[코드4] 커스텀 인증 매니저 생성
> 이 부분은 DB에서 가져오거나 외부 서버에서 사용자정보를 가져오거나 할때 적절하게 구현하면 됩니다.


### 인증 성공/실패 커스텀 핸들러
커스텀 핸들러는 선택사항이지만 특별한 처리가 필요하면 커스텀핸들러를 추가할 수 있습니다.

#### 인증성공 커스텀핸들러 생성
```java
public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    private String defaultSuccessUrl = "/";

    public CustomAuthenticationSuccessHandler(String defaultSuccessUrl) {
        this.defaultSuccessUrl = defaultSuccessUrl;
    }

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, 
                                        HttpServletResponse response, 
                                        Authentication authentication) 
            throws IOException, ServletException {
        // 성공이후 로그를 님긴다
        // 성공이벤트를 발행한다.
        // 이메일을 발송한다.
        response.sendRedirect(defaultSuccessUrl);
    }
}
````
> 커스텀 인증 성공 핸들를 생성하여 추가로직을 포함 할 수 있습니다.

#### 인증실패 커스텀핸들러 생성
```java
public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {
    private String defaultFailureUrl = "/login-error";

    public CustomAuthenticationFailureHandler(String defaultFailureUrl) {
        this.defaultFailureUrl = defaultFailureUrl;
    }

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, 
                                        HttpServletResponse response, 
                                        AuthenticationException ex) 
            throws IOException, ServletException {
        // 실패로그를 남긴다
        // 실패이벤트를 발송한다

        response.sendRedirect(defaultSuccessUrl);
    }
}
````
> 커스텀 인증 실패 핸들를 생성하여 추가로직을 포함 할 수 있습니다.


### 최종 Configuration 클래스
```java

@EnableWebSecurity
public class BrmsWebSecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/login**", "/web-resources/**", "/actuator/**").permitAll()
                .antMatchers("/admin/**").hasAnyRole("ADMIN")
                .antMatchers("/order/**").hasAnyRole("USER")
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .addFilterBefore(customAuthenticationProcessingFilter(), 
                        UsernamePasswordAuthenticationFilter.class);
    }

    // 커스텀 인증 필터
    @Bean
    public CustomAuthenticationProcessingFilter customAuthenticationProcessingFilter() {
        CustomAuthenticationProcessingFilter filter = new CustomAuthenticationProcessingFilter("/login-process");
        filter.setAuthenticationManager(customAuthenticationManager());
        filter.setAuthenticationFailureHandler(new CustomAuthenticationFailureHandler("/login"));
        filter.setAuthenticationSuccessHandler(new SimpleUrlAuthenticationSuccessHandler("/"));
        return filter;
    }

    // 커스텀 인증 매니저 
    @Bean
    public CustomAuthenticationManager customAuthenticationManager() {
        return new CustomAuthenticationManager();
    }

} 

```

### 마무리
최종적으로 `커스텀 인증 필터와 그에 필요한 클래스들의 생성 방법`을 알아보고 적용하는 방법까지 알아 보았습니다. 물론 `세부적인 내용이나 구현은 내용이 많아 생략`하였지만 원래 글의 의도인 `인증로직을 사용자에 상황에 맞게 구현`할 때 대략적인 `스프링시큐리티의 구조속에서 어떤 방향으로 접근`해야하는지에 중점을 두어 글을 작성하였습니다.

