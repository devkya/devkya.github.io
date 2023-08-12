---
title: Socket vs WebSocket
date: 2023-08-12 00:34:00 +0900
author: devkya
categories: [Django]
tags: [python, websocket, socket]
image:
  path: /assets/img/posts/thumbnails/websocket.png
  alt: preview
---

# 🙂**들어가며...**

`django` 서버를 구축하고 별 생각없이 `django-channels` 를 사용해 `WebSocket` 을 구현하여 사용하는 중이였다.

스택오버플로우 글의 추천으로 worker는 `2 x number_of_cores + 1` 로 설정하여 동시에 최대 worker 개의 작업자 스레드를 사용하게 설정했다.

잘 사용하고 있었는데…

불현듯 `WebSocket` 통신이 많아졌을 때 서버 성능이 떨어질 수도 있겠구나 생각이 들었다.

## Socket이란?

<aside>
💡 서로 다른 시스템 간에 데이터를 교환하기 위한 방법
`TCP/IP` 프로토콜을 기반으로, `Server`와 `Client`가 특정 포트를 통해 연결을 유지하고 있어 실시간으로 통신을 할 수 있는 방식이다.
특정 프로토콜(TCP, UDP)위에서 동작하며 해당 프로토콜의 규칙에 따라 데이터를 송수신함

</aside>

- **TCP(Transmission Control Protocol)**
  연결 지향적이며 신뢰성 있는 데이터 전송을 위한 프로토콜. 스트림 소켓이라고도 하며, 데이터 순서와 무결성을 보장함
- **UDP(User Datagram Protocol)**
  연결을 설정하지 않고 데이터를 전송하는 프로토콜로, 빠른 속도가 필요하고 데이터의 일부 손실이 허용되는 상황에 적합

## **Socket의 동작**

1. **소켓 생성**

   서버와 클라이언트 모두 소켓을 생성함. 이 소켓은 특정 프로토콜을 사용하여 네트워크 상의 다른 엔드포인트와 통신할 수 있는 인터페이스를 제공

2. **서버 소켓 바인딩**

   서버는 소켓을 특정 IP 주소와 포트 번호에 바인딩해야 함. 이렇게 하면 클라이언트가 해당 IP와 포트로 연결 시도할 수 있음

3. **클라이언트 연결**
   클라이언트는 서버의 IP 주소와 포트 번호를 사용하여 소켓 연결을 시도함. 연결이 성공하면 데이터를 교환할 수 있는 통신 경로가 생성됨
4. **연결 종료**

   데이터 교환 완료 후 연결 종료. 시스템 리소스가 해제되고 다른 연결을 사용할 수 있게 됨

## **WebSocket이란?**

<aside>
💡 실시간, 양방향, 전이중 통신 채널을 제공하는 고급 프로토콜
전통적인 `HTTP`와 달리 `Websocket`은 연결이 맺어진 후에도 계속 열려 있으며, 서버와 클라이언트 간에 실시간으로 데이터를 주고 받을 수 있음

</aside>

## **WebSocket의 동작**

1. **handshake**

   `Client` 와 `Server` 간 초기 연결은 `HTTP`를 사용하여 설정함. 이 단계에서 `Upgrade` 헤더가 사용되어 이후의 통신이 `WebSocket` 프로토콜로 업그레이드됨을 나타냄

2. **Data Transfer**

   연결이 열린 상태에서 양방향으로 데이터를 자유롭게 전송함

3. **Connection Termination**

   필요에 따라 연결을 닫을 수 있으며, `Server` 또는 `Client` 에서 닫을 수 있음

## **Socket vs Websocket**

### 1. 프로토콜과 레이어

- `Sokcet` : TCP 또는 UDP와 같은 전송 계층 프로토콜을 직접 사용함. OSI 모델의 `Transport Layer`에서 동작
- `WebSocket` : HTTP를 기반으로 하며, 웹 환경에서 사용하기 위해 특별히 설계된 프로토콜. OSI 모델의 `Application Layer`에서 동작

<aside>
❗ 동작 레이어의 차이는 설계와 적용 분야의 차이를 가져옴

</aside>

### 2. 연결 방식

- `Sokcet` : 장기 또는 단기 연결 설정하며, `Client` 와 `Server` 간에 데이터 송수신 가능함
- `WebSocket` : 한 번 연결이 설정되면 연결이 유지되고 양방향 통신이 가능함. 연결 유지로 인한 실시간 상호 작용이 더 쉽게 가능함

<aside>
❗ 둘 다 양방향 통신이 가능
하지만 `Socket` 은 저수준 프로토콜을 사용하므로 더 세밀한 제어가 가능하고, `WebSocket` 은 웹 환경에서 실시간 통신을 단순화함

</aside>

### 3. 사용 환경

- `Sokcet` : 일반적인 네트워크 통신과 다양한 응용 분야에서 사용 가능함
- `WebSocket` : 주로 웹 & 모바일 애플리케이션에서 실시간 통신을 위해 설계됨. HTTP와 호환되어 웹 브라우저 간의 연결을 최적화함

### 4. 보안

- `Sokcet` : 필요한 경우 TLS/SSL 과 같은 보안 프로그램을 별도로 적용해야 함
- `WebSocket` : 웹소캣 자체에 `wss://` 을 통한 보안 연결 옵션이 존재함

## **결론**

상반되는 개념이 아니기에 완전하게 비교는 불가하다. 결국에는 `WebSocket` 구현에 `Socket` 이 사용된다.

- `Sokcet` : 더 낮은 레벨의 연결을 제공하여, 저수준 제어와 성능 최적화를 위한 선택이 될 수 있음
- `WebSocket` : 웹과 모바일 환경에서 실시간 양방향 통신에 특화된 방법을 제공하며, 적용이 더 쉽고 효율적임

⇒ 내용 추가 예정…
