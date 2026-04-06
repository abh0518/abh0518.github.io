---
layout: single
title: "조잡 어플리케이션 시리즈 1 :  Android SMS Forward Application"
date: 2016-05-26 10:43:40 +0900
categories: [프로그래밍]
---

3년전 SK Planet에 입사하며 고민중 하나가 회사에서 데이터가 빵빵하게 제공되는 업무폰을 지원해준다는 것이었다. 보통들은 개인폰을 해지하고 업무폰으로 갈아탔지만 난 왠지 모르게 개인폰을 해지하기 싫어서 폰을 두개씩 달고 다니기 시작했다. 아이폰은 계속 쓰고 싶지만 그렇게 하기엔 업무폰을 안드로이드로 선택할 경우 지원되는 티스토어와 호핀 혜택이 너무 빵빵했다는 것도 이유중 하나다.
그래서 개인용 아이폰을 최저요금으로 유지하고 업무폰을 데이터 선물하기와 티스토어, 호핀 전용 머신으로 사용하게 되었다. 그렇게 사용하던 중 불편한 점을 하나 발견했는데 업무폰으로 연계되는 서비스를 사용할 때 마다 방 구석 깊숙히 충전잭에 박혀있는 폰을 꺼내서 SMS인증을 해줘야 한다는 것이다.
그래서 귀차니즘에 안드로이드용 SMS 전달 앱을 만들어 써봤다니 제법 편리해졌다. 데이터 선물하기든 호핀이든 웹이나 아이폰 앱에서 결제한 후 내 개인폰으로 전달된 SMS로 인증하면 끝이이니 이런 신세계가!

[![sms_forward_application](/assets/images/2016/05/sms_forward_application.png)](/assets/images/2016/05/sms_forward_application.png)
UI는 그낭 SMS를 전달할 번호 입력하고 save하면 끝이고 SMS는 필터링이고 나발이고 오는건 죄다 전달 해버리는 뭔가 조악한 놈이지만 아직까지 잘 쓰고 있다.

# github link : [https://github.com/abh0518/sms\_forward](https://github.com/abh0518/sms_forward "https://github.com/abh0518/sms_forward")
