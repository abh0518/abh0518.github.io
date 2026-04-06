---
title: "[Rabbitmq] 1. 초보를 위한 RabbitMQ 후딱 설치하고 설정하기"
date: 2014-10-06 18:02:05 +0900
categories: [IT기타]
---

회사의 캐시카우 서비스의 푸시 파트를 개선해야할 일이 생겼다. 레거시 코드가 너무 old한게 가장 큰 문제인데 그 중 눈에 띄는 것은 push message queue를 RDBMS로 구현해 놓은 부분이다. DB 튜닝 할때도 message pub/sub 부분에서 DB에 부하가 종종 크게 나오는 것으로 봐서는 이 부분 개선은 필수 인거 같다. 결국 mq 서버를 사용하는 것으로 결론이 났는데 무엇을 쓸까 수소문 하다가 RabbitMQ를 사용하기로 결정했다.
CloudFoundry의 Nats와 Apache ActiveMQ도 후보 중 하나였으나 난 왜 RabbitMQ를 선택했을까? 그냥 주변에서 많이 써서....
(노..농담이고 Nats는 설치도 간편하고 사용성도 좋긴 한데 클러스터링/미러링 이 안되는 가장 큰 문제점이 있다. ActiveMQ는 그냥 주변에서 RabbitMQ를 더 추천해서 써보지도 않고 팽했다.)
여튼  RabbitMQ는 처음 다루다 보니 일단 사이트(<https://www.rabbitmq.com/>) 방문해서 닥치고 매뉴얼을 읽고 Start!

## 1.  설치 (ubuntu 기반)

좀 매니악한 개발자 코스프레좀 해볼까 하고 소스 부터 설치하려했다가 erlang부터 설치해야하는 귀찮니즘의 압박으로 결국 apt-get 느님의 도움을 받기로 함
참고 페이지 : <https://www.rabbitmq.com/download.html>

```bash
$>sudo apt-get install rabbitmq-server
```

## 2. Management Plugin 활성화

설치가 완료 되면 terminal이 익숙치 않은 초보(?)를 위해 제공하는 Management Plugin도 활성화 시켜 놓자. 플러그인 설치 후에는 RabbitMQ를 restart 해주어야 변경 사항이 반영된다.
참고 페이지 : <https://www.rabbitmq.com/management.html>

```bash
$>sudo rabbitmq-plugins enable rabbitmq_management
$>sudo service rabbitmq-server restart
```

## 3. 관리자 계정  추가

처음 설치하면 guest 계정을 제공하기는 하는데 안타깝게도 localhost에서만 접속을 허용한다. 즉, 설치하자마자 Management Plugin의 유려한 UI로 막막 설정해보고 싶었지만 로그인 조차 안되는 난감함이 기다린다. 그러니 일단 RabbitMQ를 설치하고 난 뒤 바로 Terminal에서 관리자 계정을 설정해주도록 하자. 아래 예제에서는 관리자 계정을 rabbitmq로 설정했다.

```bash
$>sudo rabbitmqctl add_user rabbitmq password
$>sudo rabbitmqctl set_user_tags rabbitmq administrator
```

## 4. Management Plugin 접속

유저 설정까지 끝냈으면 Management Plugin으로 RabbitMQ를 관리해보도록 하자.
웹브라우저로 http://serverip:15672/  에 접속하면 아래와 같은 화면이 나온다.

![RabbitMQ Management 2014-10-06 17-59-02](/assets/images/2014/10/RabbitMQ-Management-2014-10-06-17-59-02.jpg)

이제 3번에서 설정한 관리자 id로 접속을 해보자. 로그인에 성공하고 아래와 같은 화면이 나오면 성공!

[![RabbitMQ Management 2014-10-06 18-00-51](/assets/images/2014/10/RabbitMQ-Management-2014-10-06-18-00-51.jpg)](/assets/images/2014/10/RabbitMQ-Management-2014-10-06-18-00-51.jpg)

축하합니다! 기본적인 RabbitMQ의 설치 및 기본 설정을 끝내셨습니다!
다음 시간엔 서비스용 계정 및 Virtual host 설정 하는 법을 진행해볼까 합니다.
