---
title : "[JPA] JPA EntityGraph"
tags : 
 - Jpa
---



# EntityGraph

쿼리 메소드마다 연관 관계의 Fetch 모드를 설정 할 수 있다.

## Fetch

* @ManyToOne의 경우 Fetch 모드 기본값이 EAGER

  EAGER의 경우 Comment 정보를 가져올때 post 정보도 미리 가져옴

* @OneToMany의 경우 Fetch 모드 기본값이 LAZY

  LAZY의 경우 post 정보가 필요한 시점에 쿼리가 날라감



##### @NamedEntityGraph

* @Entity에서 재사용할 여러 엔티티 그룹을 정의할 때 사용

  ```java
  @NamedEntityGraph(name = "Comment.post",
          attributeNodes = @NamedAttributeNode("post"))
  ```

  

##### @EntityGraph

* @NamedEntityGraph에 정의되어 있는 엔티티 그룹을 사용 해서 실제로 동작하는 로직을 구현

* 각각의 메소드 마다 다른 Fetch 전략으로 데이터를 읽어올 수 있도록 할 수 있다.

  ##### *CommentRepository*

  ```java
  @EntityGraph(value = "Comment.post", type = EntityGraph.EntityGraphType.FETCH)
  Optional<Comment> findById(Long id);
  ```

  ##### EntityGraph Type

  * EntityGraph.EntityGraphType.FETCH (default)

    설정한 엔티티 애트리뷰트는 EAGER 패치 나머지는 LAZY 패치

  * EntityGraph.EntityGraphType.LOAD

    설정한 엔티티 애트리뷰트는 EAGER 패치 나머지는 기본 패치 전략 따름

  * 기본타입들은 제외된다. ex) Long, String
