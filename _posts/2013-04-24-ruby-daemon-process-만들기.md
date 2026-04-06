---
layout: post
title: "Ruby Daemon Process 만들기"
date: 2013-04-24 11:24:56 +0900
categories: [프로그래밍]
---

==========
#!/usr/bin/env ruby
# fork로 Daemon Process 만들기
pid = fork do
Process.setsid                  # Process에 새로운 pid 할당 (fork된 Daemon이 지금의 부모 프로세스에서 떨어져 나와 최상위 프로세스의 자식 프로세스로 변경된다.)
exit if fork                         # 부모 프로세스 종료
trap("TERM") { exit }  # 시그날 설정(?), 이건 왜 쓰는지 잘 모르겠다...
*****Dir.chdir("/")                # 실행 디렉토리를 / 로 변경, Damon 프로그램끼리 파일 경로 헷갈리지 않도록 하기 위한 관례*****
# STDIN, STDOUT, STDERROR가 출력되지 않도록 /dev/null 로 출력 방향을 바꾸어 줌
*STDIN.reopen "/dev/null"***STDOUT.reopen "/dev/null", "a"*****STDERR.reopen "/dev/null", "a"***
 
# 여기서부터 Daemon으로 작동할 code 작성!
*# blablabla~~~*
end
==============
 
자세한 내용은 아래 링크에서
*http://www.jstorimer.com/2012/04/19/daemon-processes-in-ruby.html*
