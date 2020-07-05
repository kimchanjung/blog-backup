---
layout: post
title: "실시간 메시징 서버(socket.io)"
description: "실시간 메시징 서버(socket.io)"
author: kimchanjung
date: 2020-01-01 09:00:00 +0900
categories: projects
published: true
---
# 실시간 메시징 서버(socket.io)

> 멀티플랫폼(web/ios/android)을 지원하는 실시간 메시징 서버

![websocket-server-main](/post-img/projects/websocket-server/socket-server-monitor.png)
[그림 1] 메인화면

## 관련 기술블로그
- [실시간 서비스 경험기(배달운영시스템)](https://woowabros.github.io/woowabros/2017/09/12/realtime-service.html)

## 프로젝트 기간
- 2015.08 ~ 2016.08

## 프로젝트 참여도
- 프로젝트 인원 1명 
- 전담하여 개발

## 주요 기능
- websocket 실시간 메시징을 지원하여 실시간 서비스 개발을 지원함
- web/android/ios 등 멀티 플랫폼을 지원함
- 채팅 기능 구현 가능함
- 개별/채널/브로드캐스팅 등 다양한 이벤트송수신 그룹 지원
- 서버사용량/개설채널 및 클라이언트현황등의 서버모니터링 페이지 제공

## 적용된 기술 및 라이브러리
- Node.js
- Socket.io
- Redis pub/sub
- socketio-sticky-session
- Node.js Cluster

## 간략한 시스템 구성도

![websocket-server-architecture](/post-img/projects/websocket-server/socket-server-architecture.png)  
[그림 2] 구성도 

### 시스템 구성 설명
- 백오피스 서버는 API 서버와 angularjs 프론트엔드로 구성 되어 있음
- 라이더앱과 백오피스는 Socket.io로 개발된 websocket 서버를 통하여 실시간으로 배달 및 라이더 현황을 관리함
- 웹소켓 서버는 nodejs 기반 이라 싱글 프로세스로 동작하지만 서버자원을 효율적으로 사용하기 위하여 멀티프로세스로 프로그래밍하였음
- 멀티프로세스로 구동시 각각의 프로세스에 메시지를 공유하기 위해 Redis pub/sub를 사용
- 일반적으로 클라이언트가 서버 접속시 L4가 sticky session 처리를 해서 처음 접속한 서버로만 접속이 되지만 Node.js 멀티 프로세스의 경우 멀티 쓰레드가 아니므로 내부적으로 처음 접속한 프로세스에 클라이언트가 접속되도록 별도의 sticky session 처리를 해주었음


## 효과
> 실시간 이벤트가 필요한 서비스에 기능을 제공, 이를 활용하여 실시간 서비스를 실현 함에도 DB 호출 트래픽을 현저하게 낮춤

## 느낀점
- 초기 개발 문서에 간략히 기술 된 이벤트 수신후 송신 부분만 구현 하면 되는 비교적 간단한 로직으로 생각(빙산의 일각)
- Node.js는 싱글 프로세스, 서버의 CPU Core 수만큼 Multi-Processing을 코드로 구현 해야 함
- Multi-Processing로 인하여 process간 상태(이벤트 메시지 데이터 및 관리 데이터)공유 이슈는 Redis pub/sub사용하여 해결.
- Client의 접속이 처음 접속한 Process에만 통신 하도록 sticky-session을 적용하여 해결함.
- 서버현황데이터 제공을 위하여 Client, NameSpace, Room 현황 데이터를 Redis로 관리하던 중 간혹 동기식 처리로직으로 인하여 급격한 성능저하를 경험.
- Redis에는 데이터구조를 되도록 단순하게 설계, 관리 필요성과, Node.js는 동기적으로 처리하는 로직은 지양 해야함을 크게 느낌
- NameSpace, Room, EventType 별로 적절한 구조화 필요