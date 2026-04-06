---
layout: post
title: "[RabbitMQ] 2. 초보를 위한 RabbitMQ 서비스용 계정 및 Virtual Host 설정하기"
date: 2014-10-07 15:43:31 +0900
categories: [IT기타]
---

[앞장](http://abh0518.net/tok/?p=384)에서 RabbitMQ를 구동하기 위한 설치부터 기본 설정을 끝냈으니 이제 서비스가 가능하도록 계정과 Virtual Host, Queue, Exchange 설정 등을 끝내야 한다. Terminal에서 rabbitmqctl 사용하여 간지나게 작업하면 멋있겠지만 역시 초보는 초보답게 Management Plugin을 사용하여 설정하기로 함 (rabbitmqctl 공부하기 귀찮....)

## 1. 서비스용 계정 설정

관리자로 설정한 rabbitmq 아이디로 Management Plugin 에 접속해 보면 상단에 'Admin' 이라는 메뉴가 보인다.  클릭하여 Users 기능으로 계정을 관리할 수 있다.

![RabbitMQ Management 2014-10-07 14-58-08](/assets/images/2014/10/RabbitMQ-Management-2014-10-07-14-58-08.jpg)

요런 화면이 나온다. 딱히 계정관리에 어려울건 없다. GUI니 그냥 계정 두어거 생성/삭제 해보면 금방 감이 온다. 단, 주의할 것은 Tags와 Virtual Host 부분이다.

1. Tags : 게정의 권한을 부여하는 부분. Admin, Monitoring, Policymaker, Management, None 이렇게 5가지 종류가 있다. Admin은 말그대로 Rabbitmq를 마음대로 주무를 수 있는 관리자 계정이고 Monitoring은 조회만 가능하다. Policymaker,  Management 등등이 있는데 이 문서는 초보용이니 설명은 과감히 생략! **주의 사항으로******계**정의 권한을 최소한 Management 로 설정해 주어야 Management Plugin에 접속이 가능하다.** (한마디로, 지금 rabbitmq를 None으로 설정해버리면 이제 Management Plugin 사용 못한다. 으하핫!) None으로 설정해버리면 오직 Client용 계정으로 작동하여 Management Plugin에 접속이 불가능해진다. 향후 관리자용 계정(Admin), 모니터링 계정(Monitoring), 서비스 Client 계정(None)으로 분리하여 사용할때 잘 구분지어 주도록 하자.  아! Tags 라는 이름 답게 여러개 설정이 가능하다.
2. Virtual Host : Queue와 계정을 그룹핑 하는 개념이다. 사실 이것때문에 많이 방황(?)했다. 하나의 계정은 여러 Virtual Host을 할당 받을 수 있으며 자신에게 할당된 Virtual Host에 속한 Queue에만 접근이 가능하다. 자세한 설명은 뒤에서....

이미 앞선 장에서 rabbitmq 계정을 amin으로 생성했으니(그래서 Management Plugin에 접속이 가능!) 더이상 쓰지 않을 guest 계정은 삭제하고 서비스 Client 로 사용할 계정 두개를 만들어 보도록 하자. (None Tag로 pushService, smsService 두개를 만들어 보자)

## 2. Virtual Host 설정

나를 매우 혼란스럽게 해주었던 Virtual Host에 대해 알아보자! 앞서 대충 설명했듯이 일종의 Queue 접근에 대한 그룹핑이다. Exchange 란 애도 함게 설정이 되어야 하는데 일단 서비스용 계정 설정에 집중을 하도록 하자. Management Plugin에서 Admin => Virtula Hosts 메뉴에 들어가면 아래와 같은 설정 화면이 나온다.

![RabbitMQ Management 2014-10-07 15-22-28](/assets/images/2014/10/RabbitMQ-Management-2014-10-07-15-22-28.jpg)

앞에서 설정한 계정 이름을 보면 예상이 되듯이 smsService 와 pushService 두가지 서비스가 RabbitMQ를 사용할 예정이다. 이 둘은 서로 관계가 없으니 서로의 Queue나 설정을 알필요도 관심도 없다. **그러니 서비스별로 서로 독립된 구역을 나누어주는게 좋은데 그 단위로 Virtual Host를 사용할 수 있다**. smsHost와 pushHost를 당장 만들어보자. 만든 후에는 Users에서 계정별 설정을 사용하여 smsService에는 smsHost를 pushService에는 pushHost를 설정해주도록 하자. rabbitmq계정에는 모든 Virtual Host를 설정해주자. 제대로 설정했다면 Users에서 아래와 같은 화면을 볼 수 있다.

![RabbitMQ Management 2014-10-07 15-50-36](/assets/images/2014/10/RabbitMQ-Management-2014-10-07-15-50-36.jpg)

참고로 한개의 계정에는 여러 Virtual Host를 설정할 수 있다. 전체 관리자나 전체 모니터링 계정의 경우에는 모든 Virutal Host를 설정하여 쉽게 관리할 수 있도록 하는게 좋다. Virtual Host에 대한 이런 저런 Policy 설정도 가능하긴 한데 이건 초보용이니 과감히 생략! (사실 딱히 특이한 경우가 아니면 쓸일이 없을거 같다..)
다음에 Queue와 Exchange를 설정하게 되면 알겠지만 관리자 계정이든 일반 계정이든 모두 자신에게 설정된 Virtual Host 영역의 Queue와 Exchange에만 CRUD가 가능하다. 다른 Virtual Host의 Queue나 Exchange는 존재조차 확인할 수 없다.
내용은 별거 아닌데 벌써 지친다;;; 서비스용 Queue와 Exchange 설정과 개념 설명은 다음에....
