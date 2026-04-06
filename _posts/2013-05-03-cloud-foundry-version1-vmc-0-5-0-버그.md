---
layout: post
title: "Cloud Foundry Version1 VMC 0.5.0 버그"
date: 2013-05-03 10:10:43 +0900
categories: [IT기타]
---

Cloud Foundry version 1을 사용하는 중에 이상한 현상이 발생했다.
vmc로 java\_web Application을 push하면 아래와 같은 메세지가 뜨면서 앱이 아얘 업로드가 안된다.

```
Upload failed. Try again with 'vmc push'.
TypeError: can't convert nil into String
For more information, see ~/.vmc/crash
```

crash 내용따라 소스를 트래킹 해보니 vmc 0.5.0의 버그로 밝혀졌다.
정확히는 vmc 0.5.0의 버그라기보단 cfoundry 0.5.2의 버그다.
(혹시나 해서 google cloudfoundry developers groups에 물어보니 거기서도 버그 같다고 한다. 근데요즘 NG 만드느라 바빠서 신경 안쓰는거 같다.)

어쨋든 java\_web 상관없이 Ruby든 Python이든 버그를 만나면 다 푸시가 안된다.

cfoundry 0.5.2 gem의 upload\_helpers.rb를 열어보면 내부에 determine\_resources()라는 메소드가 있는데, 이녀석이 원인이다.
앱을 푸쉬하는 과정에서 Application의 크기가 upload\_helpers.rb에 정의된 RESOURCE\_CHECK\_LIMIT 보다 크면 determine\_resources() 메소드가 Application의 모든 파일을 삭제해 버린다.
왜 이렇게 만들었는지 이유는 모르겠다. @\_@

급한대로 RESOURCE\_CHECK\_LIMIT의 값을 늘려서 해결할 수 있다.
default로 64\*1024잡혀있는데 64\*4\*1024\*1024로 넉넉히 잡아주면 해결된다.

추가로, TypeError: can't convert nil into String 에러는 cfoundry 에서 압축을 하는 과정에서 나는 에러인데, 앞선 과정에서 파일이 모두 삭제되었기 떄문에 발생한 문제였다.
