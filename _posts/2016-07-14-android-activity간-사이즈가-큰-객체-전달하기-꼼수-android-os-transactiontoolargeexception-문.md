---
title: "Android - Intent extras size limit (android.os.TransactionTooLargeException) 문제 회피한 이야기"
date: 2016-07-14 18:32:23 +0900
categories: [프로그래밍]
---

안드로이드 어플리케이션 개발을 하다보면 종종 Activity나 Service에 Intent의 Extra로 데이터를 전달하는데 뭔가 애매한 경우가 있다. 그냥 Intent Extra로 ID나 쿼리 값만 전달하고 각 화면에서 필요한 시기에 데이터를 불러와도 되긴 하지만 어플리케이션 특성상 서버에서 데이터를 가져오는 경우가 많아 좋지 상황이 발생하는 경우도 있다. 특히 Activity에서 데이터를 불러오는 동안 UI들이 공백 상태로 상태로 노출되어 영 보기가 좋지 않고 불러오기가 실패했을 경우의 처리는 역시 애매하다 사실 실패의 경우는 화면마다 재시도 UI 만들기가 귀찮다.

그래서 나는 이전 화면이나 로딩화면으로 잠깐 나왔다 사라지는 Fake Activity에서 데이터를 불러온 뒤 Intent Extra로 데이터 자체를 넘겨 주는 방식을 선호한다. 불러오기를 실패했을 경우엔 그냥 다시 시도해주세요 메세지 한번 띄워주면 그만이니 실패 처리도 간단해진다. 그래서 내가 만든 어플리케이션에서는 Intent의 Extra에 데이터 객체를 담아 던져주는 경우가 많다. 물론 Java premetive type 밖에 담지 못하는 Intent의 특성상 데이터 객체는 반드시 Binary나 json으로 Serialize해야한다. serialize/deserialize 하는 비용 문제가 있긴 하겠지만 뭐 서버 사이드도 아니고 클라이언트에서 크게 문제될 일이 없었다.
한동안은 별 무리 없이 이 방법을 잘 썼다. 한가지 예외사항에 부딪히기 전까지는 말이다. 특정 상황에 데이터 객체의 크기가 생각 이상인 경우가 있었고 결국 Intent가 전달할 수 있는 사이즈를 넘어 버려 'android.os.TransactionTooLargeException' 발생하며 앱이 죽는 경우가 생겼다. 이걸 어찌하나 하고 구글링을 좀 하다가 그냥 DataHolder를 만들어 쓰면 괜춘하다는 글을 봤다.

나름 어떻게 적용해 볼까 고민하다가 아래처럼 해봤다.

1. DataHolder 클래스 생성 (둘중 뭐가 좋을지는 아직 잘 모르겠음)
   1. Application 클래스에 추가하거나....

      ```
      public class MyApp extends Application {
          private Map<String, Object> mDataHolder = new ConcurrentHashMap<>();
          
          public String putDataHolder(Object data){
              //중복되지 않는 홀더 아이디를 생성해서 요청자에게 돌려준다.
              String dataHolderId = UUID.randomUUID().toString();
              mDataHolder.put(dataHolderId, data);
              return dataHolderId;
          }
      
          public Object popDataHolder(String key){
              Object obj = mDataHolder.get(key);
              //pop된 데이터는 홀더에서 제거
              mDataHolder.remove(key);
              return obj;
          }
      }
      ```
   2. 별도의 DataHolder 스태틱 클래스를 추가하거나....

      ```
      public class DataHolder {
          private static Map<String, Object> mDataHolder = new ConcurrentHashMap<>();
          
          public static String putDataHolder(Object data){
              //중복되지 않는 홀더 아이디를 생성해서 요청자에게 돌려준다.
              String dataHolderId = UUID.randomUUID().toString();
              mDataHolder.put(dataHolderId, data);
              return dataHolderId;
          }
      
          public static Object popDataHolder(String key){
              Object obj = mDataHolder.get(key);
              //pop된 데이터는 홀더를 제거
              mDataHolder.remove(key);
              return obj;
          }
      }
      ```
2. 데이터를 보내는 측(대충 이런느낌으로?)

   ```
   BigData bigData = .......;
   Intent intent = new Intent(....);
   String holderId = DataHolder.putDataHoler(bigData);
   intent.putExtra("holderId", hoderId);
   startActivity(intent);
   ```
3. 데이터를 받는 Activity (도 대충 이런 느낌으로)

   ```
   String holderId = getIntent().getStringExtra("hoderId");
   BigData bigData = (BigData)DataHolder.popDataHolder(hoderId);
   ```

매우 간단하고 별거 아닌 꼼수인데 매우 잘 동작하고 편하고 결과도 좋다.
치명적인 단점은 당연하겠지만 Application 내에서만 작동한다는 것이다. Applicatoin간의 Intent 호출을 할 경우에는 당연히 망한다. 받는 측에서 Null Pointer Exception이 뙇! 하기도 전에 요청하는 측에서 DataHolder가 뭔지도 모르겠지....... 그러려면 DataHolder를 외부 서비스로 만들어 볼까? ...근데 그 서비스로는 데이터 객체를 어떻게 전달하나..... Intent? 그게 안되서 이렇게 한건데? 역시 꼼수는 꼼수일 뿐......

외부 전달시의 꼼수는 file로 저장한 다음 uri만 던져주는 방식으로 하면 된다고 한다. 근데 뭐 지금은 그럴일 없으니 그런 구현은 생략!
참고 사이트 : <https://www.neotechsoftware.com/blog/android-intent-size-limit>
