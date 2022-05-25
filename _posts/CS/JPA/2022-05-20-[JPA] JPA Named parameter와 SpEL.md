---
title : "[JPA] Named Parameter과 SpEL"
tags : 
 - Jpa
---



# Named Parameter과 SpEL(Spring Expression Language)

##### Named Parameter

* @Query에서 참조하는 매개변수를 1?, 2? 이렇게 채번으로 참조하는게 아니라 이름으로 :title 이렇게 참조하는 방법은 다음과 같다.

  ```java
  @Query("SELECT p FROM #{#entityName} AS p WHERE p.title = :title")
      List<Post> paramTest(@Param("title") String keyword, Sort sort);
  ```

##### SpEL

* 스프링 표현 언어
* https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#expressions
* @Query에서 엔티티 이름을 #(#entityName) 으로 표현할 수 있다.
