---
layout: post
title: "[Spring] JPA EntityListeners에서 @Autowired를 통한 의존성 주입이 미동작하는 문제 해결"
description: "@EntityListeners로 지정한 클래스에 **의존성이 필요한 경우 @autowired를 통한 의존성 주입이 동작하지 않는 문제가 있습니다 설명과 예제를 통하여 원인과 해결방법을 알아보도록 하겠습니다"
author: kimchanjung
date: 2020-06-28 20:00:00 +0900
categories: spring
published: true
---

# JPA EntityListeners에서 @Autowired를 통한 의존성 주입이 미동작하는 문제 해결
**JPA Entity 클래스에 @EntityListeners 추가** 하여 엔티티의 **CRUD가 일어날때 로그**를 남긴다던가 하는 **추가 작업**을 선언하여 **코드 중복을 줄일** 수 있습니다.  
하지만 **@EntityListeners**로 지정한 클래스에 **의존성이 필요한 경우 @autowired**를 통한 의존성 주입이 **동작하지 않는 문제**가 있습니다. 설명과 예제를 통하여 원인과 해결방법을 알아보도록 하겠습니다.

## Entity 클래스
**Rider** 엔티티 클래스에 **@EntityListeners**로 **CRUD**이벤트를 처리할 listner 클래스를 등록합니다.
```java
@Getter
@EntityListeners(RiderEntityListener.class)
public class Rider {
    @Id
    private String id;

    @Column(unique = true, nullable = false)
    private String userId;

    ...
}
```
> @EntityListeners로 지정된 RiderEntityListener 클래스에 CRUD 해당하는 액션이 일어 날때 추가 작업을 정의할 수 있다.

## RiderEntityListener 클래스
**Rider** 엔티티 **CRUD**이벤트를 처리하는 **RiderEntityListener**에 스냅샷을 저장하는 **RiderLog 앤티티**를 생성하고 저장하기위해 **RiderLogRepository의 의존성을 추가**합니다.   
**Rider**엔티티를 조작하는 코드마다 **로그를 남기는 코드의 중복을 제거**할 용도로 사용할 수 있습니다.  
```java
public class RiderEntityListener {

    private RiderLogRepository riderLogRepository;

    @Autowired
    public RiderEntityListener(RiderLogRepository riderLogRepository) {
        this.riderLogRepository = riderLogRepository;
    }

    /**
     * INSERT 이후 호출됨
     * @param riderAccount
     */
    @PostPersist
    public void postPersist(Rider rider) {
       riderLogRepository.save(new RiderLog(rider));
    }

    /**
     * UPDATE 완료 후 호출
     * @param riderAccount
     */
    @PostUpdate
    public void postUpdate(Rider rider) {
        riderLogRepository.save(new RiderLog(rider));
    }

    ...
}

```
> RiderLogRepository 클래스를 @Autowired를 통한 DI를 했지만 NPE가 발생합니다. 즉 DI가 되지 않았던 것 입니다.

## RiderLogRepository가 DI 되지 않았던 이유
**EntityListeners로 등록한 클래스**는 **Spring IOC 컨테이너의 관리 대상이 아닙**니다. 그렇기 때문에 스프링 애플리케이션이 실행되는 과정에서 **빈이 생성되고 등록되는 중**에 **EntityListeners로 등록한 클래스가 먼저 생성** 되었을 수 있습니다.   
즉 의존성 빈과 EntityListeners로 등록한 클래스가 **생성되는 시점이 서로 다른 것**입니다.
**타이밍이 잘 맞으면 의존성이 주입되어 있지만 그렇지 않으면 null인경우**가 발생하는 것 입니다.

## Spring ApplicationEvent를 통해서 분리하여 해결하는 법
이 문제와 관련하여 구글링을 해보면 **몇 가지 해결 방법**을 찾을 수 있습니다. static 변수지정하여 DI한다거나 애플케이션컨텍스트내에서 빈을 찾아 등록한다거나 **약간은 매끄럽지 못한 방법**으로 해결하는 방법을 재시합니다.  
하지만 저는 **개인적으로 ApplicationEvent를 통하여 event pub/sub 구조로 한번더 분리**하여 해결하는 방법을 추천합니다. 

### Rider 엔티티 변경 이벤트를 정의
```java
@Getter
public class RiderEntityEvent extends ApplicationEvent {

    private Rider rider

    public RiderEntityEvent(Rider rider) {
        super(rider);
        this.rider = rider;
    }
}
```
### Rider 엔티티 변경 이벤트 발행 
**ApplicationEvent** 이용하여 event를 발행합니다. **ApplicationEventPublisher가 @Autowired DI되는 이유**는 **repository**나 **service** 처럼 사용자가 생성한 빈이 아니라 **스프링이 제공하는 이미 등록된 기능의 빈**이기 때문에 **RiderEntityListener**가 생성시 **이미 ApplicationEventPublisher는 존재**하기 때문입니다.
```java
public class RiderEntityListener {

    private ApplicationEventPublisher applicationEventPublisher;

    @Autowired
    public RiderEntityListener(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }

    /**
     * INSERT 이후 호출됨
     * @param riderAccount
     */
    @PostPersist
    public void postPersist(Rider rider) {
        applicationEventPublisher.publishEvent(new RiderEntityEvent(rider);
    }

    /**
     * UPDATE 완료 후 호출
     * @param riderAccount
     */
    @PostUpdate
    public void postUpdate(Rider rider) {
        applicationEventPublisher.publishEvent(new RiderEntityEvent(rider);
    }

    ...
}
```
> 각 메소드에 이벤트를 발행하는 코드만 생성합니다.  
> 이렇게 이벤트만 발행 하도록 함으로써 추후 이 이벤트를 구독하여 로그를 남긴다거나 이메일을 보낸다거나 기타 여러가지 작업을 pub/sub형태로 개별적으로 코드를 분리하여 작성가능하므로 더 낳은 구조라고 생각합니다.  

#### 위 방법이 아니라면 아마도 이런 구조
```java
public class RiderEntityListener {
    ...

    @PostPersist
    public void postPersist(Rider rider) {
       로그저저장하기()
       이메일발송하기()
       외부API호출하기()
       //계속적인 코드추가...
    }

    ...
```
> postPersist 메소드에 수많은 기능이 추가 되겠네요....

### 발행된 이벤트를 구독하는 클래스 
**RiderEntityEvent**를 구독하여 처리하는 다수의 핸들러 클래스를 정의하고 사용합니다.
```java
// 스냅샷 저장 
@Component
public class RiderEntityLogEventHandlerImpl implements RiderEntityLogEventHandler {

    @Autowired
    private RiderLogRepository riderLogRepository;

    @Override
    @EventListener
    public void handle(RiderEntityEvent event) {
        riderLogRepository.save(event.getRider())
    }
}

// 이메일발송
@Component
public class RiderSendEmailHandlerImpl implements RiderSendEmailHandler {

    @Autowired
    private EmailService emailService;

    @Override
    @EventListener
    public void handle(RiderEntityEvent event) {
        emailService.send(event.getRider())
    }
}
```
> 이벤트를 구독하여 처리하는 클래스를 계속적으로 추가할 수 있습니다.

### 마무리
**@EntityListeners**에 등록한 클래스는 **스프링의 관리영역이 아니기** 때문에 **사용자가 생성한 빈을 사용할 때는 null** 값이 되는 경우가 있습니다.  
하지만 **ApplicationEventPublisher**를 사용하여 스프링 **이벤트 발생/구독 모델로 분리**하고 정상적으로 **사용자가 생성한 스프링빈을 사용할 수 있는 방법**을 알아 보았습니다.
