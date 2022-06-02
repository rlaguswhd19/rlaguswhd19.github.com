---
title : "[JPA] Projection"
tags : 
 - Jpa
---



# Projection

Entity의 속성이 너무 많을때 일부분만 select할 수 있는 기능

```java
select * from Commenct c;
select c.id from Comment c; <=
```



## 인터페이스 기반

* #### Closed 프로젝션

  * 쿼리 최적화를 할 수 있다. 가져오려는 속성을 정의해준다.
  * Java 8의 default() 메소드를 사용해서 연산할 수 있다. (Open 프로젝션 밑에 정리)

  ```java
  public interface CommentSummary {
      String getComment();
      int getUp();
      int getDown();
  }
  ```

  get 함수를 만들고 **CommentRepository** 만들어둔 인터페이스를 리턴값으로 함수를 만들어준다.

  ```java
  List<CommentSummary> findByPost_id(Long Id);
  ```

  그러면 쿼리가 최적화되어 원하는것만 select한다.

  ```shell
  Hibernate: 
      select
          comment0_.comment as col_0_0_,
          comment0_.up as col_1_0_,
          comment0_.down as col_2_0_ 
      from
          comment comment0_ 
      left outer join
          post post1_ 
              on comment0_.post_id=post1_.id 
      where
          post1_.id=?
  ```

  <br/>

* #### Open 프로젝션

  * @Value(SpEL)을 사용해서 연산을 할 수 있다. 스프링 빈의 메소드도 호출 가능
  * 쿼리 최적화를 할 수 없다. SpEL을 엔티티 대상으로 사용하기 때문에

  **CommentSummary**에 추가해준다.

  ```java
  @Value("#{target.up + ' ' + target.down}")
  String getVotes();
  ```

  여기서 target => comment 엔티티를 지칭하여 모든 프로퍼티를 가져오기 때문에 최적화가 되지 않는다.

  ``` java
  commentRepository.findByPost_id(savePost.getId()).forEach(c ->{
      System.out.println(c.getVotes());
  });
  ```

  ##### Query

  ```shell
  Hibernate: 
      select
          comment0_.id as id1_1_,
          comment0_.best as best2_1_,
          comment0_.comment as comment3_1_,
          comment0_.created as created4_1_,
          comment0_.down as down5_1_,
          comment0_.like_count as like_cou6_1_,
          comment0_.post_id as post_id8_1_,
          comment0_.up as up7_1_ 
      from
          comment comment0_ 
      left outer join
          post post1_ 
              on comment0_.post_id=post1_.id 
      where
          post1_.id=?
  ```

  ##### Result

  ```shell
  10 1
  ```

<br/>

* 이걸 Close 프로젝션으로 진행할 수 있다.

  ***CommentSummay***에 default 함수를 추가해준다.

  ```java
  default String getVotes() {
      return getUp() + " " + getDown();
  }
  ```

  default함수는 인터페이스에 메소드를 구현할 수 있다. 이를 구현한 메소드는 default메소드를 오버라이딩 할 수 있다. 인터페이스를 구현하는 모든 메소드를 구현해야하는 문제를 해결하기 위해서 default 메소드가 추가되었다.



#### 다이나믹 프로젝션

제네릭 타입을 줘서 리턴값을 바꿀 수 있다. 그럼 select도 마음대로 조절 가능!

```java
<T> List<T> findByPost_id(Long Id, Class<T> type);
```

