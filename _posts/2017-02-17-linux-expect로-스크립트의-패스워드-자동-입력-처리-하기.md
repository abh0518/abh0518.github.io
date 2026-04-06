---
layout: single
title: "Linux expect로 script나 command에 패스워드 자동 입력 처리 하기"
date: 2017-02-17 12:34:05 +0900
categories: [프로그래밍]
---

Ansible로 배포 스크립트를 만드는데 일부 작업에서 사용하는 모듈과 스크립트들이 종종
prompt로 패스워드나 이런저런 의사를 물어보는 경우가 있다.
한두번이야 대충 적당히 입력해 주겠는데 배포해야할 서버가 늘어날수록 여간 귀찮은게 아니다.
그래서 자동입력을 어떻게 해야하나 찾아보니 expect란 애를 쓰면 해결이 된다고 한다.

# 설치

$ apt-get install expect # 간단하네! (맥은 brew....)

# 작성 예

패스워드 프롬프트가 뜨면 파라메터로 넘긴 패스워드를 입력하고
ssh connect 연결 yes/not를 물어보는 프롬프트가 뜨면 yes를 입력해주는 예제

```
$ vi auto_password.exp
--------------------------
#!/usr/bin/expect

set timeout -1
set password 
spawn ansible-playbook -k -i $host $playbook

expect {
      "password: " { # 프롬프트에 password: 항목이 뜨는 경우 
          send "$password\r"
          exp_continue # expect 가 반복되서 처리된다.
      }
      "connecting (yes/no)?" { # 서버에 연결 할지 물어보는 경우
          send "yes\r"
          exp_continue
      }
}
--------------------------
$ chmod 755 auto_password.exp
```

실행

```
./auto_password.exp $my_password
```

# 

# 터미널 작업시

ssh 자동 접속 후 terminal 작업을 해야할때는 마지막에 interact를 붙인다.

```
#!/usr/bin/expect

set timeout -1
set password 
spawn ssh $target_host

expect {
      "password: " { 
          send "$password\r"
      }
}

interact
```

끝!
--- 추가 분 ---
