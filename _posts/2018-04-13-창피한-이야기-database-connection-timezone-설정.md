---
layout: post
title: "창피한 이야기 - Database Connection Timezone 설정"
date: 2018-04-13 15:03:59 +0900
categories: [프로그래밍]
---

요즘 Kafka로 들어오는 메세지들을 여러 저장소이 Flush해주는 데몬형태의 Java Web Service를 만들고 있다. 이 놈의 주요 기능중 하나는 Kafka메세지를 선별해서 MySQL에 저장해주는 것이다.
(그럴거면 스파크를 쓰는게 낫지 않냐 할수 있지만 관리자님께서 이리 하라 시키셨다.)

내가 Java로 Web Application을 개발할 때 고민거리 중 하나는 Database의 Timezone 설정이다. Java의 Date가 Timezone을 원활히(?) 지원하지 않다보니 Database와 Web Application의 Timezone이 다르게 설정된 경우 날짜 데이터가 꼬이기 마련이다. 특히, 그 차이를 무시하고 그냥 사용할 경우 Database의 Timestamp 기능을 사용하여 create\_time 정보를 남기면 헬게이트가 열린다. 어째 내가 Aplication찍은 시간과 Database에서 찍어준 시간이 안맞는 괴이한 현상이 발생한다. 물론 이건 괴이한게 아니라 당연한거다.
그래서 나는 Web Application과 Database의 Timezone을 항상 동일하게 맞춘다. 그럼 여러 Web Application이 동일하지 않은 Timezone을 사용할 경우 어떻게 할건데? 의문이 생기지만 그럴 경우는 Database 기준으로 시간을 조절하고 내부 로직에서 시간 변경을 하여 처리하면 된다는 식으로 넘어갔다. 아니면 Timestamp를 long type으로 Database에 저장하고 Application에서 자기 타임존에 맞춰 Date 객체로 변환해 쓰는 방식을 사용했다.
그러다가 최근 AWS로 넘어오면서 문제가 발생했다. AWS RDB서비스에서 MySQL을 사용하는데 Database의 Global Timezone 변경 권한을 주지 않는다. 그냥 UTC로 설정된 MqSQL이 나에게로 넘어왔다.

이걸 어쩌나 하고 심각하고 고민하고 있었는데, 알고보니 Database Timezone은 신경 끄고 DB Connection Session의  Timezone을 Application에 맞춰 설정하면 간단히 해결 되는 문제였다.
내가 이제 11년차 개발자인데 이걸 여태 몰랐었다니!
이 간단한걸 몰라 그동안 Database Timezone을 Wab Application과 맞추고 여러 Application에서 사용해야 할 시간 정보는 long type으로 저장해버렸던 지난 내 개뻘짓거리를 생각하니 자괴감이 밀려왔다.

어쨌든 아래와 같은 방법으로 모든 문제가 해결되었다. 그리고 머리 속 깊게 들어온 정신적 치명타는 아직 회복이 되지 않고 있다.

1. Spring Boot + Mysql 5.x 기준 DataSource 설정
   1. DB Table 과 JPA Entity 설정
      1. MySQL경우 날짜 컬럼의 Type은 TIMESTAMP여야 한다. DATETIME으로 하면 제대로 작동하지 않는것 같다. 초 이하 단위까지 남기고 싶으면 TIMESTAME(3) 으로 밀리세컨드 혹은 그 이하 까지 기록할 수 있다.
      2. JAP Entity경우 아래와 같이 컬럼 설정을 제대로 맞춰주자. 뺏을 경우 어떻게 되는지 테스트 안해봄. 닥치고 매뉴얼에서 하라는대로 하자!

         ```
         @Column(name = "log_time")
         @Temporal(TemporalType.TIMESTAMP)
         Date logTime;
         ```
   2. DataSource 설정

      ```
      @Configuration
      public class DataSourceConfig {
      
          @Bean
          @Primary
          public DataSource dataSource(){
              PoolProperties p = new PoolProperties();
              blabla...
      
              //tomcat datasource를 가져다 쓰는 이유는 묻지 말자......
              org.apache.tomcat.jdbc.pool.DataSource dataSource = new org.apache.tomcat.jdbc.pool.DataSource();
              dataSource.setPoolProperties(p);
      
              //이거 두줄이면 모든 고민이 끝나는 거였다.
              Calendar cal = Calendar.getInstance();
              dataSource.setInitSQL("set @@session.time_zone = '$TIMEZONE'".replace("$TIMEZONE", cal.getTimeZone().getID()));
              return dataSource;
          }
      
      }
      ```

DataSource 등록시 setInitSQL로 세션 타임존만 한번 선언해 주면 모든것이 해결되는 일이었다. 이제 Database timezone이 뭐로 설정되어 있던 신경 안써도 된다.
Database마다 session timezone 설정 명령은 다르니 application.yml의 db className과 함께 initial Query도 프로퍼티로 관리하면 더 편리할거 같다.
2. 사용 결과

* JPARepository : 당연히 잘된다. 한국 시간인 Application에서 현재 시간을 찍어보니면 Database에 UTC로 변환되에 들어가 있다.

  ```
  public interface JpaLogRepository extends JpaRepository<MyLog, Long> {
      List<MyLog> retrieveDigestList(@Param("ownerId") Long ownerId);
  }
  ```
* Custom Query : 넣는 데이터가 너무 많다 보니 JPARepository로 한건 한건 넣다가는 답이 나오지 않아서 그냥 Custom Query로 한번에 수십개씩 넣고 있다. 이런 경우에도 문제없이 잘 변환 되어 들어간다.

  ```
  //이건 블로그를 위해 샘플로 만든거다. 내가 업무용으로 개발한거랑은 당연 다르다.
  @Repository
  public class BulkDataRepository {
      static final Logger logger = LoggerFactory.getLogger(BulkDataRepository.class);
  
      @PersistenceContext
      EntityManager entityManager;
  
      @Transactional
      public int bulkInsertInvalidLog(List<MyLog> logs){
          if(logs.size() < 1) return 0;
  
          StringBuilder sql = new StringBuilder();
          sql.append("insert into my_log (payload, log_time) values");
          for(int i = 0 ; i < logs.size(); i++){
              sql.append("(?,?)");
              if(i != logs.size()-1){
                  sql.append(", ");
              }
          }
  
          Query query = entityManager.createNativeQuery(sql.toString());
          int position = 0;
          for(int i = 0 ; i < logs.size(); i++){
              MyLog log = logs.get(i);
              query.setParameter(1+position, log.getPayload());
              query.setParameter(2+position, log.getLogTime());
              position += 2;
          }
          return query.executeUpdate();
      }
  }
  ```

 
아 씁쓸해......
