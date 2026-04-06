---
title: "Python, datetime timezone 삽질 정리"
date: 2017-03-03 14:51:57 +0900
categories: [프로그래밍]
---

java의 DateTime은 그냥 생성해도 로컬 타임존으로 설정이 되어있어 타임존 변경이 쉬운데 파이썬은 그렇지 않다. (python version 3.5.2 기준)
그냥 datetime을 생성하면 timezone 정보가 없어 astimezone 같은 메소드를 실행하면 에러가 난다.

장고 어플리케이션 만들다가 타임존 처리를 할게 있었는데 이러한 이유 때문에 은근 삽질을 많이 했다.

결론은 datetime 생성할떄 timezone과 timedeltal를 이용해서 필요한 타임존 설정을 해주면 모든게 해결된다. (근데 귀찮다)

# Sampel Code

```
from datetime import timezone, timedelta, datetime

timestamp = time.time()
print("# timestamp를 찍어본다.")
print("time.time() => %f \n" % timestamp)

print("# timestamp를 datetime으로 변환한다.")
print("# 이 짓은 dt = datetime.now() 또는 dt = datetime.utcnow() 랑 똑같다.")
print("# 이경우 로컬 시간으로 출력 되긴 하는데 타임존 정보는 없다.")
dt = datetime.fromtimestamp(timestamp)
print("datetime.fromtimestamp(timestamp) => %s \n" % dt)

print("# datetime 생성시 timezone 정보를 넣어주면 timestamp를 해당 타임존에 맞는 시간으로 변환해 준다.")
print("# 이 짓은 dt = datetime.now(tz)랑 똑같다.")
utc_timezone = timezone.utc
dt_utc = datetime.fromtimestamp(timestamp, utc_timezone)
print("datetime.fromtimestamp(timestamp, utc_timezone) => %s " % dt_utc)
tz = timezone(timedelta(hours=9))
dt_9 = datetime.fromtimestamp(timestamp, tz)
print("datetime.fromtimestamp(timestamp, 9_timezone) => %s \n" % dt_9)

print("# 어쨋든 타임존 설정이 되어있는 datetime의 경우 쉽게 로컬 타임존을 찍을 수 있다.")
dttz = dt_utc = datetime.fromtimestamp(timestamp, timezone.utc)
print("datetime.astimezone() => %s \n" % dttz.astimezone())

print("# 결론 : datetime 생성시에 timezone 넣는걸 습관 화 하면 timezone 변환은 astimezone()으로 쉽게 사용할 수 있다.")
dt = datetime.now(timezone.utc)
print("dt = datetime.now(timezone.utc) => %s" % dt)
print("dt.astimezone() => %s" % dt.astimezone())
tz = timezone(timedelta(hours=7))
print("dt.astimezone(7_timezone) => %s" % dt.astimezone(tz))
```

# Result

```
# timestamp를 찍어본다.
time.time() => 1488520605.554720 

# timestamp를 datetime으로 변환한다.
# 이 짓은 dt = datetime.now() 또는 dt = datetime.utcnow() 랑 똑같다.
# 이경우 로컬 시간으로 출력 되긴 하는데 타임존 정보는 없다.
datetime.fromtimestamp(timestamp) => 2017-03-03 14:56:45.554720 

# datetime 생성시 timezone 정보를 넣어주면 timestamp를 해당 타임존에 맞는 시간으로 변환해 준다.
# 이 짓은 dt = datetime.now(tz)랑 똑같다.
datetime.fromtimestamp(timestamp, utc_timezone) => 2017-03-03 05:56:45.554720+00:00 
datetime.fromtimestamp(timestamp, 9_timezone) => 2017-03-03 14:56:45.554720+09:00 

# 어쨋든 타임존 설정이 되어있는 datetime의 경우 쉽게 로컬 타임존을 찍을 수 있다.
datetime.astimezone() => 2017-03-03 14:56:45.554720+09:00 

# 결론 : datetime 생성시에 timezone 넣는걸 습관 화 하면 timezone 변환은 astimezone()으로 쉽게 사용할 수 있다.
dt = datetime.now(timezone.utc) => 2017-03-03 05:56:45.554877+00:00
dt.astimezone() => 2017-03-03 14:56:45.554877+09:00
dt.astimezone(7_timezone) => 2017-03-03 12:56:45.554877+07:00
```
