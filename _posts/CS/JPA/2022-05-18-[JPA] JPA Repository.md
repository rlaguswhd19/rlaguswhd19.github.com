---
title : "[JPA] JPA Repository"
tags : 
 - Jpa
---



## JPA Repository

@EnableJpaRepositoies

* 스프링 부트 사용할 때는 사용하지 않아도 자동 설정 됨.
* 스프링 부트 사용하지 않을 때는 @Configuration과 같이 사용



@Repository 안붙여도 된다. JpaRepository에 이미 붙어있기 때문에

스프링 @Repository

SQL Exception 또는 JPA 관련 예외를 스프링의 DataAccessException으로 변환 해준다.



##### Example

```java
@Test
public void createPost() {
    Post post = Post.builder()
            .title("test")
            .build();

    postRepository.save(post);

    List<Post> posts = postRepository.findAll();
    assertThat(posts.size()).isEqualTo(1);
}
```

Post의 Id에 @GenratedValue를 빼고 id없이 넣는 경우 에러가 발생한다.

`org.hibernate.id.IdentifierGenerationException` => `hibernate`에서 발생한 Exception

`org.springframework.orm.jpa.JpaSystemException` => `hibernate`에서 발생한 Exception을 `Spring` Exception으로 변환



Spring의 DataAccessException 계층 구조의 목적은 SQLException 하나로 모든 에러를 판단하기 힘들기 때문에 안에 코드값에 따라 DataAccessException의 하위 클래스중의 하나로 매핑하여 이름만 봐도 알 수 있게 한다.

그러나 hibernate의 예외 계층 구조가 잘 만들어져 있다.
