---
title : "[JPA] Query by Example"
tags : 
 - Jpa
---



# Query by Example

QBE는 필드이름을 작성할 필요 없이 (X 프로퍼티를 작성해야 한다.) 단순한 인터페이스를 통해 동적으로 쿼리를 만드는 기능을 제공하는 사용자 친화적인 쿼리 기술입니다.

Example = Probe + ExampleMatcher

* Probe는 필드에 어떤 값들을 가지고 있는 도메인 객체
* ExampleMatcher는 Probe에 들어있는 그 필드의 값들을 어떻게 쿼리할 데이터와 비교할지 정의한 것
* Example은 그 둘을 하나로 합친것, 이걸로 쿼리를 함

##### CommentRepositoryTest

```java
@Test
public void qbe() {
    Comment probe = Comment.builder()
            .best(true)
            .build();

    ExampleMatcher exampleMatcher = ExampleMatcher.matchingAny()
            .withIgnorePaths("up", "down", "likeCount", "created");

    Example<Comment> example = Example.of(probe, exampleMatcher);

    commentRepository.findAll(example);
}
```

기본값이 있는 up, down, likeCount, created는 Ignore해서 쿼리를 만든다.



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
    where
        comment0_.best=?
```

<br/>

장점

* 별다른 코드 생성기나 애노테이션 처리가 필요 없음

* 도메인 객체 리팩토링 해도 기존 쿼리가 깨질 걱정하지 않아도 됨 => X

  필드 이름을 작성해야 하므로 깨짐

* 데이터 기술에 독립적인 API

단점

* nested 또는 프로퍼티 그룹 제약 조건을 못 만든다.
* 조건이 제한적이다. 문자열은 start/contains/ends/regex가 가능하고 그밖에 property는 값이 정확히 일치해야 한다.



QueryByExampleExecutor
