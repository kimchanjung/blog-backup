---
layout: post
title: "[Socket.io][websocket] 실시간 서비스 경험기(배달운영시스템 BROS1.0)"
description: "websocket 웹소켓 실시간 채팅 socket.io"
author: kimchanjung
date: 2020-05-08 00:00:00 +0900
categories: tech
published: true
---

## 경험기

들어가기 앞서 이 글은 신기술 사용기 또는 소개가 아닌 실시간 서비스 즉 **배민라이더스 BROS 1.0** 을 개발 하면서 겪어왔던 다소 특별한 개발 및 운영 경험기 입니다. BROS 2.0이 나온 상황에서 1.0을 이야기 하는 것이 다소 순서가 맞지 않지만 그 때 당시 경험기를 남겨놓지 못 한 것을 이번 기회에 남겨 보고자 합니다.

## 용어설명

### BROS 1.0

BROS는 배민라이더스의 주문을 접수(콜센터접수)를 시작으로 배달건을 라이더가 고객에게 신속하고 안전하게 배달하기 위한 통합 운영 관제 시스템입니다.

### REAL-TIME 또는 실시간 서비스

여기서 Real Time 또는 실시간서비스란 **websocket**으로 통신하는 서버를 이용하여 웹페이지 및 모바일 앱에서 실시간으로 데이터를 주고 받고 갱신하는 시스템을 말합니다.

### 양방향바인딩

**AngularJS**의 양방향 바인딩을 뜻 하며 뷰의 데이터가 변하면 모델이 자동으로 변경되고 반대로 모델이 변하면 뷰도 자동으로 변경되는 특징을 말합니다.

### Digest Loop

AngularJS에서 모델 변화를 감지하는 역할을 하여 양방향 데이터 바인딩을 적용하는 역할을 합니다. AngularJs1.0의 성능적인 면에서 단점으로 작용하기도 하지만 개별 모델의 **one-time bind** 설정을 통하여 양방향 바인딩이 필요없는 모델을 제외 할 수 있습니다. 

### [Websocket](https://developer.mozilla.org/ko/docs/WebSockets/Writing_WebSocket_client_applications)

웹브라우저는 http 프로토콜로 요청/응답으로 동작하는데 TCP/IP socket 처럼 connection이 유지되어 서로 실시간으로 통신을 할수 없습니다. 그래서 등장한것이 ws<sup>websocket</sup> 프로토콜입니다.  websocket을 사용하면 웹브라우저에서도 socket 통신처럼 실시간으로 데이터를 주고 받을 수 있습니다. 최근에는 대부분의 브라우저가 websocket 프로토콜을 지원하지만 IE 같은 경우는 version 10 부터 지원을 하고 있습니다.

### [Socket.io](https://socket.io/)

 **NodeJS** 기반으로 실시간 이벤트 서버를 개발 할 수 있는 오픈소스 라이브러리 입니다. 특징으로는 멀티 디바이스(web, android, ios, windows)를  지원하며 **websocket**을 지원하지 않는 Browser도 지원합니다.

## 시스템구성

* **PHP API server**
* **AngularJS(1.0) front-end**
* **Socket.Io (NodeJs websocket open source library)** 
* **Redis**
* **Sql server(MS)**


## 실시간 서비스가 필요했던 이유

배민라이더스의 음식 주문은 평균 1시간 내외로 주문의 접수, 처리 및 배송이 완료 되어야 합니다. 즉 주문이 발생한 경우 **콜센터직원은 바로 주문의 발생을 알아야** 하며, 주문접수가 처리 되면 **라이더는 즉시 배달건의 존재를 알아야** 합니다. 또한 **배달의 상태(대기 - 배차 - 픽업 - 전달)와 라이더의 실시간 위치가 업데이트** 되어야 관제자와 라이더들은 원할한 관제 및 배달업무를 수행 할 수 있습니다. 또한 기존 polling 방식의 배민 콜센터 주문접수(현재는 fade out)는 주문건이 증가함에 따라 DB Select가 급증하여 서비스가 위험했던 적이 있었기  때문에 BROS개발 당시 실시간성을 유지하면서 DB Select를 줄이기 위해서 필수로 실시간 이벤트 서버 도입을 해야 했습니다.

![order-activity-diagram](/post-img/real-time-service/order-delivery-activity.png)

그림 1. 주문 처리 activity 다이어그램

## AngularJS 양방향 바인딩과 Socket.io를 이용한 실시간 데이터 동기화

주문과 배달의 생성 상태변경이 있을 때 마다 socket.io 실시간 이벤트를 전송하고 수신 시 api를 호출하여 배달리스트를 갱신하는 방식은 데이터의 변경이 있는 경우만 database를 select하지만 피크시간대에는 이벤트가 증가하므로 database select가 급증하게 됩니다. **angularjs model** 변경은 angularjs가 알아서 뷰에 반영하기 때문에 실시간 이벤트를 송수신 할 때 마다 배달리스트를 호출 하지 않고 배달데이터를 **생성 삭제 수정 한 후 실시간 이벤트 메시지로 angularjs model에 반영** 하도록 하면 뷰는 **자동으로 실시간 반영** 됩니다. 또한 개발자는 뷰를 업데이트하는 비즈니스로직을 신경쓸 필요가 없고 데이터를 뷰에 나타나는 로직만 구성해 놓으면 됩니다.

![angularjs-model](/post-img/real-time-service/angularjs-model.png)

그림 2. AngularJs 모델 데이터 반영

### 1.Controller - 실시간 이벤트 수신

```javascript
angular.module('bros').controller('bros.dlvrymgmt.common.DlvryMgmtCommCtrl', [
  'bros.dlvrymgmt.common.realTimeDlvryService',
  'bros.dlvrymgmt.common.Delivery','SktIo', '$scope',
    function(realTimeDlvryService, Delivery, SktIo, $scope) {
      	// 배달데이터 객체를 모델에 binding 이제 뷰와 모델은 양방향 바인딩이 된다!!
      	$scope.delivery = Delivery
        /**[소켓이벤트수신] 배차 생성 (상태는 대기와 동일)*/
        SktIo.on('delivery:create', function (err, delivery) {
            realTimeDlvryService.onCreate(err, delivery);
        });
        /**[소켓이벤트수신] 배차대기*/
        SktIo.on('delivery:waiting', function (err, delivery) {
            realTimeDlvryService.onWaiting(err, delivery);
        });

       ........
```
코드 1. 이벤트 수신 Controller

###  2.Service - 수신받은 데이터를 배달 객체에 반영

```javascript
angular.module('bros').factory('bros.dlvrymgmt.common.realTimeDlvryService', [
    'bros.dlvrymgmt.common.Delivery','$resource',
    function(DlvryMgmtCommModel, Delivery){
        var realTimeDlvryService = {};
      	/**신규배달 처리*/
      	realTimeDlvryService.onCreate = function (err, delivery) {
            Delivery.of(delivery);
        };
        /* 배차대기 처리*/
        realTimeDlvryService.onWaiting = function (err, delivery) {
            Delivery.setResource(delivery);
        };
      	return realTimeDlvryService;
}]);
```
코드 2. 수신이벤트 처리 Service

### 3.Model - 배달 모델 객체를 변경

```javascript
angular.module('bros').service('bros.dlvrymgmt.common.Delivery', ['$filter',
    function($filter) {
        var Delivery = this;
        var instance = {};
        /**객체 초기화*/
        Delivery.init = function () {
            return angular.copy(instance, Delivery);
        };
        /**배달객체생성*/
        Delivery.of = function (delivery) {
            angular.extend(Delivery, delivery);
        };
        /**배달데이터 변경 반영*/
        Delivery.setResource = function(delivery) {
            $filter('setResource')(Delivery.items, delivery);
        };
        angular.copy(this, instance);
    }
]);
```

코드 3. 수신받은 데이터 모델에 반영

> 데이터의 **생성, 업데이트, 삭제**를 **Database에 반영**하고 **곧바로 Socket.io 서버의 실시간 이벤트 메시지**로 데이터를 전송 **Angular Model에  반영**, 뷰는 모델의 변경에 **자동 갱신** 되기 때문에 **client수에 관계없이** 사실상 Database를 주기적으로 **select하는 행위는 거의 일어나지 않으며**  배달데이터는 실시간으로 관제 및 라이더에게 반영된다.

### 배달현황 화면

![bros-delivery-list](/post-img/real-time-service/bros-delivery-list.png)
그림 3. BORS 실시간 배달현황 화면

### 라이더 관제 및 배차 화면

![bros-gps](/post-img/real-time-service/bros-gps.png)
그림 3. BORS 실시간 라이더 배차 화면

## 맞닥뜨려야했던 이슈들

**AngularJS**와 **Socket.io** 서버를 이용하여 DB select를 최소화 하고 주문, 배달 및 라이더 위치를 실시간으로 관제하고 배차업무를 할 수 있도록 시스템을 구성하고 실시간으로 눈앞에 데이터들이 화면 갱신 없이 변경이 되고 있었지만 가만히 화면만 바라보고 있을 때는 큰 문제가 없는 듯 하였습니다.

*"그러나 browser는 열일 하고 있는 중…..."*

## Browser의 성능

실시간으로 화면이 렌더링 되는 것은 사실 아무런 문제가 없었습니다. 하지만 피크시간인 저녁시간에 배달 건수가 급격히 늘어나기 시작하면서 배차를 위해 라이더리스트 다이얼로그를 클릭한다던지 하는 이벤트를 발생 시켰을 때 0.5초 정도 움찔하는 Delay가 발생하였습니다. 사용자 입장에서는 꽤나 신경이 거슬리는 이슈였습니다.

> 요구사항으로 배달완료된 배달건도 리스트에 남기고 당일 모든 배달건(많을 때는 1000건)을 현황에 리스팅 해달라는 요구가 있었습니다. 물론 타협으로 최근 4시간 배달건만 리스팅 하기로 하였지만 피크시간대에는 수백건의 배달을 페이징없이(실시간리스트라 페이징은…) 리스팅 되고 있었습니다.

일반적인 초기 화면 진입 이후 뷰의 렌더링이 거의 없는 정적인 페이지와 달리 BROS의 배달/라이더 현황 페이지들 JAVASCRIPT가 실시간으로 이벤트를 수신받고 모델에 반영하고 뷰를 렌더링하고…… 실 새 없이 일을 하고 있었습니다. 그래서 성능최적화 작업에 들어갔습니다.

### 모든 loop를 native for로 변경

**forEach, angular.foreach**등으로 된 loop를 제거하고 **순수 javascript의  역행 루프**<sup>reversed loop</sup>로 변경하였습니다. 배열의 개수가 적을 때는 크게 상관 없지만 수백 수천개가 되면 그때는 이야기가 다릅니다.

```javascript
// length 지역 변수에 미리 선언(매번확인하지 않는다), 루프 순서를 역으로 
for (var i = delivery.length-1; i >= 0; i--) {
  	// for문 내부에 배열을 지역변수에 할당
    var deliveryItem = delivery[i]
    deliveryItem.deliveryStatus = "pickup";
};
```
코드 4. 역행 루프

* for (var i = delivery.length-1; **i >= 0; i--**) 루프 순서를 역으로 하면  조건문을 0 즉, false로 평가하게 하여 속성검색(조건문 비교)을 최소화하는 효과를 얻는다.
* for (**var i = delivery.length**;i >= 0; 배열의 크가룰 마리 할당 해놓아 배열을 크기를 매번 확인하지 않는다.
* **var **deliveryItem** = delivery[i]** 배열을 미리 지역변수에 할당 해놓고 deliveryStatus 연산시 참조 하면  delivery[i].status, delivery[i].shopName 처럼 매번 배열을 검색하지 않는다. 

### setTimeout을 사용하여 로직을 큐로 넘긴다.

javascript는 **단일 쓰레드로 동작**하며. 먼저 수행된 작업이 끝날 때 까지 다음작업은 대기하게 된다. 무거운 작업이 있다면 당연히 사용자는 Delay를 느끼게된다. 이러한 점을 해결하기위해 **setTimeout을 이용하여 작업을 실행하면 javascript engine에서 UI 작업 큐로 작업은 넘겨** 지게 되고 event loop가 **큐의 쌓여 있는 task를 처리** 하게 됨으로써 좀더 **blocking이 감소**하여 좀더 성능향상을 시킬 수 있다. [JAVASCRIPT Event Loop 링크](https://github.com/nhnent/fe.javascript/wiki/June-13-June-17,-2016)

```javascript
 setTimeout(function () {
   Rider.updatePosition(position);
 },0)  
```

코드 5. setTimeout 사용

### Angular ng-repeat loop 사용시  조합 index key를 사용.

**ng-repeat**은 콜렉션을 looping하여 뷰에 리스팅 하는 angularjs의 커스텀 attribute입니다. 리스팅된 데이터는 digest loop가 양방양 데이터 바인딩을 위하여 관리하는 모델 데이터 들이며 데이터 **개수가 많을 수록 digest loop의 성능이 떨어지게** 됩니다. 그래서 리스트의 각 항목 업소명, 배달상태, 배달주소.. 등등의 데이터는 **onetime binding으로 digest loop를 가볍게** 하고 **배달번호+수정일시가 조합된 index key**의  **변경 감지만**으로 **뷰를 자동 갱신**하게 됩니다.

```
 <tr ng-repeat="deliveryItem in delivery | track by (deliveryItem.deliverySeq+deliveryItem.modDate)">
    <td ng-bind="::deliveryItem.shopName"></td>
    <td ng-bind="::deliveryItem.status"></td>
    <td ng-bind="::deliveryItem.riderName"></td>
 </tr>  
```
코드 6. 조합 index key

* <td ng-bind="::deliveryItem.shopName"></td> 나머지 모델데이터는 onetime binding 처리


### App 처럼 화면에 나타나는 리스트만 렌더링

v-repeat이라는 angularjs용 오픈소스 모듈을 사용하여 scroll up & down 할때 화면에 나타나는 tr을 렌더링하고 사라지는 tr은 제거 되도록 처리 하여 리스트가 개수가 수백 수천이 되더라도  **화면에 보이는 데이터 모델만 존재 하게 되어 실질적으로 digest loop가 관리하는 모델의 개수가 현저하게 줄어드는 효과**를 낼 수 있었고 실제로 **성능 향상에 제일 큰도움**이 되었습니다.

### [Web Worker](https://developer.mozilla.org/ko/docs/Web/API/Web_Workers_API/basic_usage) 사용을 고려 하였으나 결국은 미적용...

javascript 실행을 메인쓰레드가 아닌 백그라운드쓰레드에서 처리하게 할 수 있게 하여 무거운 작업의 경우 백그라운드 쓰레드가 처리하도록 하여 기존 단일쓰레드에 비해서 성능향상을 이점을 얻을 수 있습니다. 

angularjs의 digest loop를  web worker가 처리 하도록 하고 싶었으나 digest loop를 건드리는 일은 angularjs framework core를 건드리는 일이 되어 버리므로 결국 적용을 포기하였습니다 (가장 아쉬운 부분이기도 합니다.)

## 실시간 이벤트 서버의 안정성

사용료를 지불하고 바로 사용할 수 있는 *유료 서비스들이 존재합*니다. 대표적으로 **[pubnub](https://www.pubnub.com/)**, **[pusher](https://pusher.com/?gclid=EAIaIQobChMIkdbaubyY1gIVmAgqCh0OTwKSEAAYASAAEgLVY_D_BwE)** 같은 서비스가 대표적이며 **websocket서버를 직접 개발할 필요없이  사용 할 수 있는 장점이** 있습니다. 반면에 장애나 이슈 발생시 즉각적인 처리가 어렵다는 단점도 분명 존재합니다. 실제로 BROS 2.0도 유료 서비스를 사용하다. 장애나 오류 발생시 즉각적인 대응이 어려워 결국은 websocket 서버를 개발하여 사용하고 있습니다.

***"유료서비스를 사용할 것인가 직접 만들것 인가는 여건과 상황에 따라 판단은 달라 질 수 있습니다."***

### 실시간 이벤트 유실

websocket server는 client와 서버 간에 http protocol로  커넥션을 초기에 맺고 ws<sup>websocket</sup>protocol로 upgrade한 후 서로에게 heartbeat를 주기적으로 발생시켜 커넥션이 유지되고 있는지 체크하며 네트워크를 유지합니다.

socket.io를 사용하여 websocket 서버를 개발 했지만 비즈니스로직 문제가 아닌 **다양한  network 상황 때문에 이벤트 유실이 발생** 했습니다. 발생하는 **건수는 매우 적은 수준**이였지만 BROS 서비스의 특성상 *1건이라도 누락이 발생하면 배달업무에 차질*이 생기기 때문에  필수적으로  이벤트 유실에 대한 보완이 필요했습니다 . (**유료 서비스도 마찬가지로 발생하는 이슈**)

이벤트 유실을 보완하기 위해 **[RabbitMq](https://www.rabbitmq.com/)같은 메시지 큐를 사용하여 이벤트를 발송하는 것도 고려** 하였으나 BROS 서비스의 특성상 시간이 지난 이벤트를 수신 받게 되거나 **한참이 지난후 한꺼번에 미수신된 이벤트를 수신받게 되면** 잘못된 데이터가 반영될 수도 있는 문제가 발생하게 되고 그 **문제해결을 위해 복잡한 로직을 추가하게 되면 오히려 파생되는 문제가 더 생길 것으로 판단**하였고 되도록이면 근본적인 해결책을 찾기로 하였습니다. 

### 브로드 캐스팅

cron job을 사용하여 2분에 1번씩 batch proccess 한곳에서 만 배달 데이터를 select 하여 database  부하를 줄이면서socket.io 실시간 이벤트로 브로드캐스팅을 하도록 하고 client는 수신받은 데이터로 유실이 발생한 배달 리스트를 fetch 하는 것으로 이벤트 누락에 대한 데이터 미변경을 보완 하였고 적용 이후에는 사용성에 대한 문제가 보고 되지 않았습니다.(더 좋은 방법도 분명 있을 겁니다.)

![namespace-room-id](/post-img/real-time-service/websocket-broadcasting.png)

그림 4. 이벤트 유실을 보완하기 위한 Broadcasting

### Websocket 서버 다운상황에 대비

어느 순간이나 서버가 다운되면 안되지만 만약에 다운이 된다면 심각한 장애를 초래하게 됩니다. 실시간 서비스를 개발한다면 항상 염두해 두어야 하는 이슈 입니다. BROS1.0은 socket.io server가 disconnect가 되면 바로 api 직접 호출로 변경이 되고

설정해둔 주기만큼 reconnection 시도 하도록 되어 있으며 reconnection이 성공하면 api 직접호출은 중단키시고 실시간 이벤트수신으로  swiching 되도록 개발 되어 있습니다. 더좋은 방법은 메소드들을 추상화 하고 2개이상의 실시간 이벤트 서버를 switching 할 수 있으면 더욱 안정적인 시스템이 될 수 있을 것이란 생각도 해봅니다.

## 실시간 이벤트 서버<sup>socket.io</sup>

이제 실시간 서비스를 위해 필수 적으로 필요한 websocket서버에 대한 이야기를 해보고자 합니다. 앞서 잠시 언급 하였지만 **서버자체를 구축할 필요없이 유료로 이용할 수 있는 서비스들이 많이 존재**합니다. 유료서비스의 장점은 **개발과 운영에 대한 리소스가 들지않고**  사용할 수 있다는 장점이 있습니다. 하지만 **이슈 발생시 빠른 대처가 어렵다는 단점도 분명 존재**하구요. 직접 서버를 개발하거나 유료 서비스를 이용하거나 하는 **선택은 여러가지 상황에 따라 판단**해야 할 듯 싶습니다. 그리고 서버를 **직접 개발 하고 안정적인 상태로 유지하기 까지 생각보다 기술 적인  learning curve  높은 편이며 서버가 안정적인 상태까지 올라오기 위해서는 실제로 운영을 해봐야 한다는 어려움도 존재**합니다. Socket.io 서버를 **개발 하면서 겪었던 여러가지 경험에 대해서 이야기 하려고 합니다.** 이야기할 내용은 Socket.io 서버에만 국한된 이야기라기 보다 **websocket 서버를 개발한다면 아마도 동일하게 겪어야 될 경험**이라고 생각됩니다.

## 용어설명

### Namespace & Room & Event

socket.io에서 트래픽을 격리하여 구분하는 단위로 사용됩니다 event는 명칭 그대로 송/수신하는 이벤트의 이름입니다. 트래픽격리 구분없이 이벤트를 송/수신하면 이벤트 리스너를 등록하여 이벤트를 처리하는 코드가 없더라도 접속한 모든 client에 전송 및 수신을 하게 됩니다. 불필요한 트래픽이 발생하게 되고 서버 자체의 성능도 저하되기 때문에 적절한 설계로 구분해아합니다.

![namespace-room-id](/post-img/real-time-service/namespace-room-id.png)


표 1. Socket.io 트래픽 격리 구분 

> 다른 서비스에서는 room이란 용어 대신 channel이라는 용어를 많이 사용합니다.

### Public & Private & Broadcasting

socket.io에서 이벤트를 송/수신하는 방식을 말합니다.

![public-private-broadcasting](/post-img/real-time-service/public-private-broadcasting.png)

표 2. Socket.io 이벤트 송수신 방식

### Cluster

NodeJS는 기본적으로 싱글 프로세스로 동작하며 서버 CPU Core 수 만큼 proccess 생성하여 Multi Proccess로 구동하기 위해서는 Cluster를 이용하여 Proccess를 생성하게 됩니다.

> **multi thread**는 thread간 데이터를 공유되지만 multi processing은 데이터공유가 되지 않는 특징이 있다.
>
> 그래서 특별한 처리들을 구현해야 한다.

### Master & Worker

**NodeJS Cluster** 를 이용하여 Proccess를 생성하면 실제 일을 수행하는 proccess를 **Worker** 라고 하며 worker들을 제어하는 역할을 하는 proccess를 **Master**라고 부른다.

## Socket.io를 선택한 이유

socket.io는 앞서 말한것 처럼 **websocket을 지원하지 않는 브라우저도 지원**합니다. websocket을 지원하지 않는 브라우저에서는  flashsocket, htmlfile, xhr-polling, jsonp-polling등의 **적절한 방식으로 전환되어 통신**합니다. 최근에는 버전업이 되면서 websocket, polling만 지원하는 것으로 변경 되었습니다.

> flashsocket같은 경우는 브라우저에 flash가 설지되어 있지않으면 작동을 하지 않는 문제가 있었고 안정성이 떨어지는 방식은 지원에서 제외 되었습니다.

NodeJs 특징인 **Single Thread 기반의 Non-Blocking I/O으로 성능적인 이점**이 있습니다 (callback 지옥이라는 단점도 있지만..)

## 자 이제 개발 해보자…..

socket.io 홈페이지에 문서를 보니 아래 처럼 간단한 것 같은 ….. 금방 만들수 있겠다...

> socket.io document는 detail함이 좀 부족한 느낌이다. 그리고 버전업이 되면서 deprecated 메소드나 설정 값들이 많아 혼란 스러워 socket.io object들을 실제로 console.log로 찍어서 객체를 확인 하면서 개발 해야 했다 휴…...

```javas
// 접속되면
io.on('connection', (socket) => {
  // 이벤트를 수신받고	
  socket.on('say to someone', (id, msg) => {
    // 보내면 되는 구나..
    socket.to(id).emit('my message', msg);
  });
});
```

코드 7.  Server측 코드

```javascript
//수신 받고
socket.on('news', (data) => {
  //데이터 처리
});
// 보내고
socket.emit('hello', 'world');
```

코드 7.   Client측 코드

> 원래 호수에 있는 오리를 보면 편하게 둥둥 떠있는 것 처럼 보인다. 하지만 물속은 난리다…..

물론 기본 설정이나 여러가지를 구현해야 하지만 큰 로직은 수신/송신이라 할 것이 많이 없을 줄 알았다.(websocket 서버를 쌩으로 구현하는 예제들도 기본예제들이긴 하지만..) 하지만 실제로 실무 서비스에 사용하려고 하니 여러가지 고려 대상이 생각보다 많았다. 이제부터는 실제로 개발 하면서 겪었던 과정을 설명 하려고 한다.

## 트래픽의 격리와 전송방식을 잘 구현 해주어야 한다(꼬옥~)

BROS는 각 **지역 마다 센터로 구분** 되어서 일을 하기 때문에 **이벤트의 송수신은 센터 끼리** 해야 된다. 그리고 같은 센터라도 배달데이터 송/수신을위한 Room과 채팅을 위한 Room은 구분 되어야 한다. 전체 센터의 배달 현황을 봐야 할 경우도 있다. 같은센터의 Room이지만 배달 상태를 전달완료로 변경하기 위해서는 Event명을 지정하여 송/수신 할 수 있어야 한다. 특정 Client에게만 선택해서 이벤트를 전송하기 위해서는 private로 전송 해야 하며 채팅의 메시지는 public이 되어야 한다.

그래서 서버는 namespace/room/event 트래픽격리 구분과  public/private/broadcasting 이벤트 전송 방식을 실무에서 사용할 서버라면 필수로 구현해야 한다.

## Scale out에 대한 대비

여기서 부터 ***맨붕***이 왔다. 일반 웹서버처럼 세션관리만 신경쓰면 Server를 스케일아웃 하더라도 사람이 신경쓸 것이 별로 없는 상황이 아니였다. **nodejs는 싱글 프로세스라  멀티프로세스를 생성하고 서로 완벽한 Clustering**을 해주어야 했다. 추후 **서버 자체의 Scale out이 되었을 경우에도 대비해야 하므로 Clustering을 구성하는 것은 꼭 필요**했다. 단순히 세션에 대한 문제 뿐만 아니라 1번 서버 > 1번 프로세스에 접속된 Client가 이벤트를 전송하면 나머지 서버 > 나머지 프로세스들에 접속된 클라이언트로 이벤트를 전송하기 위해서 **프로세스 끼리는 데이터가 연결되어 전/수송이 되게 작업**이 되어야 했다.

### Clustring

nodejs는 싱글 프로세스라 node cluster로 core 수만큼 프로세스를 생성해야 했다. 중요한 것은 멀티 쓰레드 방식이 아닌 **멀티 프로세스방식이라 데이터 공유가 되지 않는 문제**가 있었고 **데이터 공유에 대한 처리**가 필요했다.

```javascript
if (cluster.isMaster) {
     //CPU의 갯수만큼 워커 생성
    os.cpus().forEach(function (cpu) {
        cluster.fork();
    });
    //워커가 죽으면,
    cluster.on('exit', function(worker, code, signal) {
        if (code == 200) {
            //워커를 살리고 살리고~~
            cluster.fork();
        }
    });

    //생성한 워커가 보내는 메시지 처리
    worker.on('message', function ("야 김찬정이가 접속했단다!") {
        //생성한 워커에게 메시지 보내기
    	worker.send("워커들아 김찬정이가 접속했다는 구나!");
    });
}

if (cluster.isWorker) {
    //마스터에게 메시지 보내기
    process.send("야! 여기 김찬정이가 접속했다 다른 워커에게 좀 알려줘라 마스터야! 1명 접속 추가!!");
    
   //마스터가 보낸 메시지 처리
    process.on('message', function("김찬정이가 접속했네!") {
        // 김찬정이가 접속 했으니 카운트도 추가하고~ Redis에 계정이랑 조인한 룸정보도 저장하고....
        // 어휴... 번거롭기 짝이.....
    });
}
```

코드 8. nodejs cluster

위 코드 8 처럼 인위적으로 cpu 수만큼 woker를 생성 하고 워커들 끼리 통신 하기 위해서는  Master에게 메시지를 보내고 다시 나머지 Worker에게  데이터를 전송한다. ("*아.. 뭔가 framework이 알아서 해주는게 아니라 사람이 코드로 저렇게 해줘야 하다니… 시람은 언제나 실수를….*")

### Clustering 시 데이터 공유는 Redis Pub/Sub

**worker 끼리 이벤트를 전/송수신 하는 매개체로 redis pub/sub 를 이용**했다. worker 1에 접속된 client가 이벤트를 전송하면 나머지 worker들에게  redis pub/sub을 통해 이벤트를 전송하고 수신받아 자신에게 접속된 client들에게 최종적으로 이벤트 메시지를 전송한다.

![socketio-clustring](/post-img/real-time-service/socketio-clustring.png) 

그림 5. 클러스터링 구성도

### Worker의 Sticky Session 처리

스케일 아웃 된 서버에 client가 접속할 때 마다 서버를 달리하여 접속하게 되면 세션문제를 마주하게 된다. 그래서 일관성있게 지정된 서버에만 접속 되도록 Sticky Session을 보통 L4같은 로드밸런서 장비가 해주게 되는데. 물리적인 서버는 로드밸런서가 처리해 준다고 하지만 문제는 node cluster로 생성한 worker들이다. 1대의 물리 서버에 worker들이 멀티프로세스로 동작하는 것은 사실 서버가 여러대 돌아가는 것이나 마찬가지 상황! worker들의 Sticky Session 처리도 오픈소스 모듈을 사용에 처리해주었다.(NodeJs 같은 특별한 경우가 아니면 필요 없을 수도….)

## ws<sup>websocket</sup>가 아닌 http 이벤트 송신 api도 필요!

Client는 Server와 websocket으로 connection을 유지하고 서로 통신하지만 http로 이벤트를 전송할 수 있는 기능도 필 수로  필요합니다.  **client가 websocket으로 연결되어 있을 필요는 없고 event 발송만 하면 되는 경우도 필요**하고 수신은 websocket으로 송신은 restful api 처리 끝단에 http로 이벤트를 전송하는 방식으로 시스템을 구성 할 수도 있기 때문에 http 이벤트 전송 api도 구현 해야할 필요가 있습니다. 실제로 BROS 에서 콜센터 주문접수처리 하기 위해서 주문의 상세 화면을 보고 있는 경우 고객이 **배민앱에서 주문 취소를 하게 되면 주문취소 api 끝단에 http로 주문 취소 이벤트를 송신**했고 콜센터 주문접수 화면에서는 바로 고객주문 취소 안내를 표시 했습니다.

```javascript
// POST - http://webmsg.woowa.in/ridersorda로 요청이 들어오면
app.get('/ridersorda', function (req, res, next) {
  next();
}, function (req, res) {
  // POST로 전송된 값들을 가져오고
  var nameSpace = req.param('nameSpace');
  var roomName = req.param('roomName');
  var rcvEvtNm = req.param('rcvEvtNm');
  var rcvData = req.param('rcvData');
  
  // websocket 이벤트로 전송
  io.of(nameSpace).to(roomName).emit(rcvEvtNm, rcvData);
  
  // http response 설정을 해주고 끝!
  res.set('Content-Type', 'application/json; charset=utf-8');
  res.status(200).send("{'status':'ok','message':'성공'}");
  res.end();
});
```

코드 9. 이벤트 송신 http api

## 서버를 띄어 놓긴 했다만 관리는 어떻게하나...

서버를 만들어서 띄어 놓긴 했지만 서버에 대한 관리가 필요했다.**(이런 부분때문에 유료 서비스를 사용하는 것이..)** 일단 서버에 접속된 **Namespace/Room별 접속한 클라이언트 현황**이 필요 했고. **서버의 cpu/memory 등의 정보**등이 필요했다. 그리고 서버에서 **오류가 발생했거나 멈췄을 경우 즉각적인 Notification**이 필요했다.

### 서버 현황 데이터들은 Redis에 저장 관리

앞서 말한 것 처럼 **접속한 Client의 수와 이름 과 Room 리스트 데이터들은 각각의 worker 프로세스에서 공유 되지 못하고 따로 관리가 되기 때문에 이런 관리 데이터들은 1곳에 저장하고 조회 할 필요성**이 생겼다. 또한 **채팅 기능에서 서로 간 대화 메시지들이 보관되어야 할 저장소**도 필요했다. Clustering 구성시에 Redis를 사용하고 있었기 때문에 Redis를 활용하여 데이터들을 저장하고 사용했고 Client들이 접속 및 Room에 join/out 하거 할때 Redis에 정보를 update했고 실시간 이벤트로 모니터링 페이지에 전송했다.

![socket-server-monitor](/post-img/real-time-service/socket-server-monitor.png)

그림 6. 소켓 서버 현황 페이지

### 오류가 발생하면 이메일로 바로 오류 내용 전송

사실 이부분은 유료 APM 서비스를 사용한다면 바로 해결될 부분이지만 (개발 당시는 사용하지 않았다.. 역쉬 돈이 쵝오!) 그렇지 않는 경우 필수로 오류상황에 대한 피드팩을 즉각적으로 받아야 할 필요가 있다. socket.io server를 개발 할 당시에는 winston 모듈을 사용하여 error 레벨의 로그가 남겨지거나 exception이 발생 했을 경우 설정해놓은 이메일로 바로 전송 되도록 했다.

## Socket 서버에서 발행 했던 이슈들

개발하면서 다양한 문제들을 접했는데 client 수가 낮은 데도 cpu 100%로 서버마비를 겪은 적도 있고 redis는 메모리 저장소이지만 메모리데이터를 파일에 백업을 하는데 백업옵션에 문제가 생겨오류가 발생 했던 일, IE 브라우저에서 접속한 Client가 창이 닫혔을 경우 disconnected를 서버가 인지하지 못한 경우등등을 이야기 하고자 한다.

### 서버 CPU 100%

클라이언트가 많지 않은 상황이였는데 서버접속이 되지 않았다. 서버가 죽은 것은 아니였지만 이상하게 CPU 100% 상태였고 원인은 바로 NodeJs 특징인 Single Thread 기반의 Non-Blocking I/O 에서 비동기 이벤트 loop의 가장 적인 sync한 로직 처리 때문에 발생했다. 로직을 sync 하게 처리하게 되면 로직이 완료 될 때가지 block이 발생하게 되고 다음 처리 로직은 대기하게 된다(비동기 처리의 자세한 내용은 [링크클릭](http://www.nextree.co.kr/p7292/)) 비동기 처리를 한 이유는 Redis에서 namespace/room/clientAccount 등의 정보를 저장해놓고 가져올 때 처리한 로직 때문이였다. **nodejs 비동기 이기 때문에 for loop가 일반적인 동작순서로 수행되지 않는다.**

```javascript
a = [1, 2, 3, 4, 5]; 
for (var i=0; i<a.length; i++) {
    someFunction(function(){
    	console.log(i)               
    });
}
// 예상결과
// 1,2,3
// 실제결과
// 3,3,3 이건 뭥미?
```

코드 10. nodejs loop 동작

이러한 특징을 해결하기 위해 async 모듈을 사용하여 for loop를 sync 하게 처리 하도록 했지만 결국 문제를 일으키고 말았다.

```javascript
 async.waterfall([ function(waterfallCB) {
   // 1.모든 클라이언트를 가져오고
   redisClient.keys('SocketIo^All_Client^*', function(err, nspsarr) {                                              		async.forEachOf(nspsarr, function (nspsVal, nspsKey, forEachOfCB) {                                                   
       redisClient.hgetall('SocketIo^All_Client^'+room, function(err, object) {             
         forEachOfCB();  
       });
     }, function (err) {
	     waterfallCB(err);  
     });   
   });
}, function(waterfallCB) {
  // 2.모든 접속 계정을 가져오고
  redisClient.hlen('SocketIo^Account', function(err, count) {
      allsocketInfo.connectCnt = count;
      waterfallCB(count); 
  }); 
}, function(waterfallCB) {
  // 3. 모든 socket id를 가져온다음
  redisClient.keys('SocketIo^SocketId^*', function(err, socketIds) {
      allsocketInfo.accountCnt = socketIds.length;
      waterfallCB(count); 
  }); 
}] ,function(err, data) {
  // 적절하게 조합한 데이터를 만든 후 리턴
  allsocketInfo.clientInfo = data;
  allsocketInfo.roomCnt = allsocketInfo.roomList.length
  callback(err, allsocketInfo);
});
```

코드 11. 중첩 loop sync 하게 처리하기 위해 async라는 모듈 사용

redis에 저장된 서버 현황 데이터를 가져와서 적절하게 조합하기 위해 for loop 가 여러번 사용 되었는데 **async라는 모듈을 사용하면  loop가 수행된 후 결과를 순차적으로 전달함으로 써 sync 하게 처리** 할 수 있었지만 사용률이 증가하면서 **성능문제가 발행**했고 문제의 해결은 

1. 되도록 async모듈을 사용한 loop를 처리하지 않는다.**(sync하게 처리하여 block상황을 만들지 않는다)** 
2. redis의 데이터 구조를 단순화 한다.

*redis는 string, list, set, hash 등의 key/value 구조의 데이터 타입을 지원하므로 RDB 스타일의 복잡한 계층구조로 데이터를 저장하게 되면 데이터를 가져와 조합하기 위해서 복잡한 처리를 하게 되므로 처음 부터 제공되는 데이터 타입에 잘 맞추어 데이터를 설계하고 저장하면 불필요한 for loop문을 줄일 수 있다.*

### Redis 파일저장 오류

socket 서버의 namespace/room/account/socketid 관리 데이터들은 서버의 현황 데이터기 때문에 사실상 file로 백업 될 필요가 없지만 Chatting시 채팅에 입장 후 재입장 시 대화 내용을 다시 보여 줘야 했기 때문에 redis가 재부팅 되거 나 하는 경우에도 영구적으로 기록 되어 있어야 했다.

![bros-chatting](/post-img/real-time-service/bros-chatting.png)

그림 7. 채팅기능

*redis에서 파일 저장 시  여러가지 옵션들이 있는데 **stop-writes-on-bgsave-error** 옵션이 **yes**로 되어 있는경우 파일로 저장하다가 오류가 발생하면 redis의 메모리에 데이터 저장 자체가 안되게 되서 오류를 발생 시킨다 **no** 변경하게 되면 메모리에 저장하는 행위는 파일 저장 오류과 관계없이 계속 수행하게 된다.* 

이처럼 redis의 옵션에 따른 예상치 못한 오류때문에 socket server에 오류가 발생 하였고 옵션변경으로 문제를 해결했지만 redis 서버자체의 옵션에 대한 정보도 알아놓을 필요가 있다.

### IE에서 disconnect를 인식 하지 못하는 문제

IE에서 Browser 창을 닫는 경우 **서버에서 client가 disconnect된 것을 인식 하지 못해 서버측에서 해당 client의 disconnect 처리를 하지 못 하였고 서버 측 disconnect 이벤트에 구현 되어 있던 데이터 처리가 제대로 되지 않아 서버 현황데이터에서 client가 살아 있는 것으로 표시** 되었다. client sdk에서 세팅값을 추가하면 해결되는 문제 였다. 여기서 꼭 짚고 넘어가야 할 부분이 있다. 유로 서비스 또는 오픈소스 라이브러리들은 기본적으로 client sdk를 제공하는데 물론 오픈소스 라이브러리는 서버를 개발 해야 하지만 기본적으로 둘다 **안정적으로 connection 관리를 해주고  서버주소, 옵션값 connection, reconnection, connection interval, error 핸들링, 이벤트 전송/수신 등등의 메소드를 제공**합니다. 개발자는 제공하는 메소드들만 잘 사용하면 됩니다.

***javascript WebSocket 객체가 onopen, onclose, onerror, onmessage 등의 기본 메소드들을 제공하지만***  WebSocket 객체로 client를 *쌩*으로 개발 하게 된다면 **connection 관리가 불안하거나 장애를 경험하면서  안정적이 되어 가는 과정을 겪을 수 있을 수도 있습니다.** 선택은 각자의 몫이지만 실시간 이벤트 서버를 사용해야 한다면 **최대한 안정성이 높은 유료서비스를 선택 하고 차선은 오픈소스라이브러리를 사용**하여 개발 "쌩"으로 개발 하는 것은 다시한번 고려 해보아야 됩니다...



## 끝으로..

실시간 서비스를 개발 하게 된다면 **중요하게 고려해야될 부분은 Browser의 성능이다** 일반적인 웹페이지에서는 그렇게 와닫는 문제는 아니지만 **실시간 서비스는 반드시  Browser의 성능과 마주하게 된다**. 그리고 **실시간 이벤트 서버를 직접 개발 할 것인가 유료서비스를 사용할 것 인가를 결정하는 것**이다 **서로 장단점을 충분히 고려 해서 최대한 안정적인 방향으로 결정**해야한다 그렇지 않으면 **"배달건이 안보여요~"라는 피드백을 자주 들을 것이다** ㅠㅠ

BROS 1.0을 개발 하면서 처음 마주하는 framework와 기술들을 사용하면서 어려움도 있었지만 **무척 흥분된 상태로 일했었다.** 퇴근 하면 거의 매일 코딩했고 주말에도 거의 대부분 코딩했다.  회사를 다니면서 실무를 하면 **현실적으로 이런 경험을 하기는 쉽지 않지만 몇가지 상황이 충족이 되었기 때문이였던 것 같다.**  일단 오프라인(라이더센터와 사업적인)이슈들에 더 집중 되면서 다음 개발 이슈들에 대한 **기간적인 여유**가 약간 있었고, **처음 접해보는 신기술?(그때 당시에는)**로 개발 하면서 흥미가 높았다. 또한 **개발에 집중 할 수 있도록 챙김이님의 배려**의 가 있었기 때문이 였다. (늦게 나마 다시한번 회사와 동료들에게 감사 드립닷~~)














