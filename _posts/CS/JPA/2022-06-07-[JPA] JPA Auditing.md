---
title : "[JPA] Auditing"
tags : 
 - Jpa
---



# Auditing

스프링 데이터 JPA Auditing

`Audit`은 감시하다라는 뜻으로 Spring Data JPA에서 시간에 대해서 자동으로 값을 넣어 주는 기능이다. audit을 이요하면 자동으로 시간을 매핑하여 데이터베이스 테이블에 넣어준다.

##### Comment.class

```java
@EntityListeners(AuditingEntityListener.class)
public class Comment {

    @CreatedDate
    private LocalDate created = LocalDate.now();

    @ManyToOne
    @CreatedBy
    private CommentAccount createBy;

    @ManyToOne
    @LastModifiedBy
    private CommentAccount updateBy;

    @LastModifiedDate
    private LocalDate updated;
}
```

@EntityListeners(AuditingEntityListener.class) 추가한다.

##### Application.class

```java
@EnableJpaAuditing(auditorAwareRef = "commentAccountAuditAware")
public class SpringJpaApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringJpaApplication.class, args);
    }

}
```

Application에 넣어주는데 위에 코드중에 updateBy, createBy의 Account를 누군지 알수가 없다. 그래서 유저정보를 시큐리티에서 가져오는 AuditorAware 인터페이스의 구현체를 만들어야한다.

##### CommentAccountAuditAware.class

```java
@Service
public class CommentAccountAuditAware implements AuditorAware<CommentAccount> {

    @Override
    public Optional<CommentAccount> getCurrentAuditor() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (null == authentication || !authentication.isAuthenticated()) {
            return null;
        }
        return ((MyUserDetail) authentication.getPrincipal()).getUser();
    }
}
```

빈의 이름은 앞글자만 소문자로 바꾸면 된다. 그리고 Application위에 `@EnableJpaAuditing(auditorAwareRef = "commentAccountAuditAware")`를 추가해준다.

<br/>

## JPA 라이프 사이클 이벤트

어떠한 엔티티에 변화가 일어날때 특정한 callback을 실행할 수 있는 이벤트를 발생시켜준다.

entity에 정의할수 있다.

```java
@PrePersist
public void prePersist() {
    System.out.println("JPA event call back");
	this.created = LocalDate.now();
}
```



@PrePersist 엔티티 저장하기 전에 호출

@PreUpdate 엔티티 업데이트하기 전에 호출

https://www.baeldung.com/jpa-entity-lifecycle-events
