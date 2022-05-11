---
title : "[JPA - SpringDataCommon] 쿼리 만들기"
tags : 
 - Jpa
 - SpringDataCommon
---



# 쿼리 만들기

스프링 데이터 저장소의 메소드 이름으로 쿼리 만드는 방법

1. 메소드 이름을 분석해서 쿼리 만들기 (CREATE)

2. 미리 정의해 둔 쿼리 찾아 사용하기 (USE_DECLARED_QUERY)

   적용순서 : @Query > @Procedure > @NamedQuery

3. 미리 정의한 쿼리 찾아보고 없으면 만들기 (CREATE_IF_NOT_FOUND) => default

   ```java
   public interface CommentRepository extends MyRepository<Comment, Long> {
       @Query(value = "SELECT c FROM Comment AS c", nativeQuery = true)
       List<Comment> findByTitleContains(String title);
   }
   ```

<br/>

##### 메소드 이름으로 쿼리 만들기(CREATE)

> 리턴타입 {접두어}{도입부}By{프로퍼티 표현식}{조건식}{And,Or}{프로퍼티표현식}{조건식}{정렬조건}{매개변수}



| 접두어          | Find, Get, Query, Count, Delete                              |
| --------------- | ------------------------------------------------------------ |
| 도입부          | Distinct, First(N) Top(N)                                    |
| 프로퍼티 표현식 | Person, Addresss.ZipCode => find(Person)ByAddress_ZipCode(...) |
| 조건식          | IgnoreCase, Between, LessThan, GreaterThan, Like, Contains, ... |
| 정렬 조건       | OrderBy{프로퍼티}Asc\|Desc                                   |
| 리턴 타입       | E, Optional<E>, LIst<E>, Page<E>, Slice<E>, Stream<E>        |
| 매개변수        | Pageable, Sort                                               |

<br/>

##### Example

```java
List<Comment> findByCommentContains(String comment);

Page<Comment> findByLikeCountGreaterThanAndPost_IdOrderByCreatedDesc(int likeCount, Long postId, Pageable pageable);

Page<Comment> findByLikeCountGreaterThanAndPost(int likeCount, Post post, Pageable pageable);
```

<br/>

##### 확인

제대로 만들었는지 궁금하다. 그럼 그냥 돌려보면 된다. Repository를 Bean으로 등록할때 쿼리를 만들기 때문에 에러가 발생한다.
