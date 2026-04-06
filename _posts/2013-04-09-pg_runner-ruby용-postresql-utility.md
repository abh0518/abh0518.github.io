---
title: "pg_runner - Ruby용 Postresql Utility"
date: 2013-04-09 22:17:22 +0900
categories: [프로그래밍]
---

Ruby로  백그라운드에서 postgresql 작업을 하는 데몬 프로그램을 만드는 중인데, Ruby on rails의 Active Record에 쓰기에는 너무 오바인거 같고 Single Connection으로 만들기에는 뭔가 아쉽고 불안하다.
여기저기 뒤져보다가 딱히 맘에드는 connection pool이 없어서 연습삼아 만들어 봤는데, 생각보다 잘돌아간다. (속도는 아직 보장 못함)  Pool로 사용할 Synchronized Array를 루비에서 지원해주는지 알길이 없어 여기저기 뒤져보다 마음을 무겁게 만드는 영문의 압박에 그냥 비슷한 기능하는 Wrapper 클래스를 만들어 썻다. 다행이 이것도 생각보다 잘돌아 간다. (Array Wrapper에 Mutex만 걸어준거니 안돌아 가는게 이상하겠지만. @\_@)
현재 버전에서 아쉬운점은 transaction기능이 없고 postgresql 전용이라는거...

다음에 시간날때 추가해봐야겠다.
요번에 만드는 모듈에 이거 넣어야지 헤헤헤헤....
궁금하신 분들은 (있을까 모르겠지만 ) 다음 링크를 참조하시길...
<https://github.com/abh0518/pg_runner>
