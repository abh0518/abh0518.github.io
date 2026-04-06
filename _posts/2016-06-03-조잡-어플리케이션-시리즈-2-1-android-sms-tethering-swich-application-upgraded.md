---
layout: single
title: "조잡 어플리케이션 시리즈 2-1 : Android SMS Tethering Swich Application Upgraded"
date: 2016-06-03 14:07:41 +0900
categories: [프로그래밍]
---

[조잡 어플리케이션 시리즈 2 : Android SMS Tethering Swich Application](http://abh0518.net/tok/?p=480 "조잡 어플리케이션 시리즈 2 : Android SMS Tethering Swich Application") 의 마지막에 이야기 했던대로 '꺼' 명령 날리는 것 조차 까먹는 일이 비일비재 해졌다. 인간의 선천적 게으름과 망각은 어쩔 수 없나보다.
아침마다 꺼진 폰을 보고 망연자실 하는 것도 한두번이지 이럴바에 그냥 Keep Alive 시간을 체크하는 Timer를 넣어 보기로 했다. 그리고 하는 김에 자체적으로 테더링을 on/off 할 수 있도록 해봤다. (영어가 개판인건 무시합시다.)

![activity_main.xml - sms_tethering_switch - [~:AndroidStudioProjects:sms_tethering_switch] 2016-06-03 13-52-21](/assets/images/2016/06/activity_main.xml-sms_tethering_switch-AndroidStudioProjectssms_tethering_switch-2016-06-03-13-52-21.png)

별거 없다. 전과 비교해서 화면 상단에 테더링을 on/off 할수 있는 토글 버튼 하나 추가 되었고 하단 부분에 keep alive 설정이 추가되었다. keep alive에 분 단위로 시간을 써주면 해당 시간이 지난 후 테더링을 자동으로 꺼준다. AlarmManager를 쓰면 되는 정말 별거 아닌 기능이다.
이제 아침마다 테더링으로 인해 배터리가 다 날라간 핸드폰을 마주하는 일이 없겠지?
github link : [https://github.com/abh0518/sms\_tethering\_switch](https://github.com/abh0518/sms_tethering_switch "https://github.com/abh0518/sms_tethering_switch")
