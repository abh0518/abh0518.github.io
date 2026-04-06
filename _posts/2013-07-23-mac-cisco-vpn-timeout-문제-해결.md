---
title: "Mac Cisco VPN Timeout 문제 해결"
date: 2013-07-23 10:45:54 +0900
categories: [IT기타]
---

Mac (Mountain Lion)에서 Cisco VPN연결을 하면 1시간 마다 패스워드를 물어보면서 사람을 무지 귀찮게 한다.
/var/run/racoon 에 있는 설정파일을 약간 수정하야 해결할 수 있든데, 자세한건 다음 링크 참조.
Disconnects 항목부터 참조하면 된다.
[http://anders.com/guides/native-cisco-vpn-on-mac-os-x/](http://anders.com/guides/native-cisco-vpn-on-mac-os-x/ "http://anders.com/guides/native-cisco-vpn-on-mac-os-x/")
