---
title: "Python, Decorator 써본 이야기"
date: 2017-02-01 19:42:35 +0900
categories: [프로그래밍]
---

어쩌다 python-django 로 웹 어플리케이션 하나를 만들게 되었는데 이거 은근 신세계다. django 같은 프레임워크야 뭐 많으니까 놀랄게 없더라도 python의 decorator는 나에게 뭔가 신세계를 보여줬다. java-spring에서 그렇게 복잡했던 AOP가 decorator를 쓰면 그냥 별 고민 없이 끝나버린다. 이걸 AOP라고 해도 될런지 모르겠지만 말이다. 여튼 decorator에 감탄한 나머지 까먹기 전에 decorator 파트만 정리해본다.

# 일반 데코레이터

1. 이렇게 코딩하고 실행하면

   ```
   # 얘가 데코레이터
   def decorator(func):
       def decorator(*args, **kwargs):
           print("%s %s" % (func.__name__, "before"))
           result = func(*args, **kwargs)
           print("%s %s" % (func.__name__ , "after"))
           return result
       return decorator
   
   # 함수에 데코레이터를 붙여준다.
   @decorator
   def func(x, y):
       print(x + y)
       return x + y
   
   func(1,2)
   ```
2. 이런 결과가 나온다. 아! 신통방통 하다!

   ```
   func before
   3
   func after
   ```
3. @데코레이터는 사실 이거랑 같은 의미라고 한다

   ```
   def decorator(func):
       def decorator(*args, **kwargs):
           print("%s %s" % (func.__name__, "before"))
           result = func(*args, **kwargs)
           print("%s %s" % (func.__name__ , "after"))
           return result
       return decorator
   
   def func(x, y):
       print(x + y)
       return x + y
   
   func2 = decorator(func)
   func2(1,2)
   ```

# 파라메터를 가지는 데코레이터

1. 데코레이터에 뭔가 파라메터를 전달하고 싶을데는 약간 복잡하긴 하지만 역시 다 된다! function을 감싸는 decorator를 다시 감싸주면 된다.

   ```
   # 얘가 파라메터도 붙는 데코레이터
   def decorator_with_param(param):
       def wrapper(func):
           def decorator(*args, **kwargs):
               print(param)
               print("%s %s" % (func.__name__, "before"))
               result = func(*args, **kwargs)
               print("%s %s" % (func.__name__ , "after"))
               return result
           return decorator
       return wrapper
   
   @decorator_with_param("hello, decorator!")
   def func(x, y):
       print(x + y)
       return x + y
   
   func(1,2)
   ```
2. 결과는 이렇게 나온다!

   ```
   hello, decorator!
   func before
   3
   func after
   ```

# 그런데 func.\_\_doc\_\_이 나오지 않는다. 아,  망했다!

1. 원랜 이렇게 나와야한다. 그래야 Swagger UI 같은애랑 붙일때 자동으로 문서화가 된다.

   ```
   def func(x, y):
       """
       x와 y를 더합니다.
       :param x:
       :param y:
       :return:
       """
       print(x + y)
       return x + y
   
   print(func.__doc__)
   
   ---- 출력 ----
   
    x와 y를 더합니다.
    :param x:
    :param y:
    :return:
   ```
2. 그런데 데코레이션을 붙이는 순강 망한다. \_\_doc\_\_이 안나온다. 실제로 django api application을 만들면서 api endpoint 메소드들을 decorator로 신나게 감쌌더니 Sweager UI에서 doc 처리하지 못해 공백 API 가이드만 한가득 나왔다.

   ```
   def decorator(func):
       def decorator(*args, **kwargs):
           print("%s %s" % (func.__name__, "before"))
           result = func(*args, **kwargs)
           print("%s %s" % (func.__name__ , "after"))
           return result
       return decorator
   
   @decorator
   def func(x, y):
       """
       x와 y를 더합니다.
       :param x:
       :param y:
       :return:
       """
       print(x + y)
       return x + y
   
   print(func.__doc__)
   
   ---- 출력 ----
   None # 아, 망했어요!
   ```
3. 생각해보면 당연한 일이다. 실행시간에 실제로 접근하는 메타데이터는 func가 아니라 데코레이터가 만들어준 wrapper의 메터데이터니 제대로 나올리가 없다. 그렇다. 우린 망했다.
   <iframe width="560" height="315" src="https://www.youtube.com/embed/frEm8dftIoI" frameborder="0" allowfullscreen></iframe>
4. 그렇다고 진짜 망한건 아니다. decorator에 @wraps 달아주면 모든것이 해결된다. 모든 decorator에는 반드시 @wraps를 달아주자. 그것이 모두가 행복해지는 길이다. 이유는 찾아보기 귀찮아서 생략. (대충 소스 보니 func의 \_\_doc\_\_ 같은 meta 정보를 wrapper에 복사해 넣는거 같은데 확실한건 아님!)

   ```
   from functools import wraps
   
   # 파라메터 없는 데코레이터에도 @wraps 붙여주고
   def decorator(func):
       @wraps(func)
       def decorator(*args, **kwargs):
           print("%s %s" % (func.__name__, "before"))
           result = func(*args, **kwargs)
           print("%s %s" % (func.__name__, "after"))
           return result
       return decorator
   
   
   # 파라메터 있는 데코레이터에도 @wraps 붙여주고
   def decorator_with_param(param):
       def wrapper(func):
           @wraps(func)
           def decorator(*args, **kwargs):
               print(param)
               print("%s %s" % (func.__name__, "before"))
               result = func(*args, **kwargs)
               print("%s %s" % (func.__name__ , "after"))
               return result
           return decorator
       return wrapper
   
   
   @decorator
   def func(x, y):
       """
       x와 y를 더합니다.
       :param x:
       :param y:
       :return:
       """
       print(x + y)
       return x + y
   
   
   @decorator_with_param("hello, decorator!")
   def func2(x, y):
       """
       x와 y를 더합니다.
       :param x:
       :param y:
       :return:
       """
       print(x + y)
       return x + y
   
   
   print(func.__doc__)
   func(1, 2)
   print(func2.__doc__)
   func2(1, 2)
   ```
5. 실행하니 잘 나오네!

   ```
     x와 y를 더합니다.
    :param x:
    :param y:
    :return:
    
   func before
   3
   func after
   
    x와 y를 더합니다.
    :param x:
    :param y:
    :return:
    
   hello, decorator!
   func2 before
   3
   func2 after
   
   ```

# class로도 decorator 선언이 가능하다고도 합니다.

class로 만드는게 뭔가 낙타표기도 되고 그래서 뭔가 그 뭔가 멋져보이는거 같은데 여기엔 치명적인 단점이 있다. @wraps를 붙일수가 없다. 그나마 parameter를 가지는 데코레이터의 경우 \_\_call\_\_ 시점에서 wrapper를 만들면서 @wraps를 붙여줄수 있는데 parameter가 없는 데코레이터의 경우 wraps를 붙일 방법이 보이질 않는다. 이거 저거 찾아보니 결국 \_\_doc\_\_, \_\_name\_\_들을 복사해 넣는데 이럴거면 그냥 function으로 데코레이터 만들란다.

```
from functools import wraps

# 그냥 Class 데코레이터, @wraps를 붙일만한데가 보이지 않는다.
class Decorator:
    def __init__ (self, func):
        self.func = func

    def __call__ (self, *args, **kwargs):
        print("%s %s" % (self.func.__name__, "before"))
        result = self.func(*args, **kwargs)
        print("%s %s" % (self.func.__name__, "after"))
        return result


# 파레매터를 가지는 Class 데코레이터, __call__에서 @wraps를 넣어준다.
class DecoratorWithParam:
    def __init__ (self, param):
        self.param = param

    def __call__ (self, func):
        @wraps(func)
        def decorator(*args, **kwargs):
            print(self.param)
            print("%s %s" % (func.__name__, "before"))
            result = func(*args, **kwargs)
            print("%s %s" % (func.__name__, "after"))
        return decorator


@Decorator
def func(x, y):
    """
    x와 y를 더합니다.
    :param x:
    :param y:
    :return:
    """
    print(x + y)
    return x + y

@DecoratorWithParam("hello, decorator!")
def func2(x, y):
    """
    x와 y를 더합니다.
    :param x:
    :param y:
    :return:
    """
    print(x + y)
    return x + y


print(func.__doc__)
func(1,2)
print(func2.__doc__)
func2(1,2)

---- 출력 ----
None # 아, 이거 짜증나네
func before
3
func after

 x와 y를 더합니다.
 :param x:
 :param y:
 :return:
 
hello, decorator!
func2 before
3
func2 after
```

끝!
