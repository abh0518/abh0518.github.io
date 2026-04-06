---
title: "Python, Django Model을 main 루틴에서 에서 사용하기(?), Django Model을 서버 구동 없이 사용하기(?)"
date: 2017-03-03 15:37:37 +0900
categories: [프로그래밍]
---

django로 어플리케이션 만들면서 종종 귀찮은 DB 작업들을 해야할 떄가 있는데 (데이터 삭제, 변경 등등...) DBA가 아니다보니 SQL작성이 매우 귀찮다.
게다가 django덕분에 ORM에 익숙해져 버리니 가뜩이니 사이 안좋은 SQL과는 더욱 사이가 나빠져 있는 상태가 되어 있다.
그래서 난 python 코드로 Django Model을 돌려 DB작업을 하기로 했다.

# Sample

```
import os
import django
from django.db import transaction

# django setting 파일 설정하기 및 장고 셋업
cur_dir = os.path.dirname(__file__)
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "my_application.settings")
django.setup()

# 모델 임포트는 django setup이 끝난 후에 가능하다. 셋업 전에 import하면 에러난다. db connection 정보가 없어서......
from my_application.models import MyModel

@transaction.atomic
def update_my_model_data():
    datas = MyModel.objects.all()
    for data in datas:
        # 하고 싶은거 하고 
        data.save()

if __name__ == "__main__":
    update_my_model_data()
```

# 결론

SQL 안써도 되서 행복해요.
