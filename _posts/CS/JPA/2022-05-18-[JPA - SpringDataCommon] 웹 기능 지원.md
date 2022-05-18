---
title : "[JPA - SpringDataCommon] 웹 기능 지원"
tags : 
 - Jpa
 - SpringDataCommon
---



## SpringDataCommon 웹 기능

##### 제공하는 기능

* 도메인 클래스 컨버터

* @RequestHandler 메소드에서 Pageable과 Sort 매개변수 사용

* Page 관련 HATEOAS 기능 제공

  * PagedResourcesAssembler

  * PagedResource

```java
@GetMapping("/posts")
public ResponseEntity<PagedModel<EntityModel<Post>>> getPost(Pageable pageable, PagedResourcesAssembler pagedResourcesAssembler) {
    Page<Post> page = postRepository.findAll(pageable);
    return ResponseEntity.ok(pagedResourcesAssembler.toModel(page));
}
```



##### Pageable Request

```
/posts?sort=id,desc&page=0&size=10
```

<br/>

## DomainClassConverter

```java
@GetMapping("/posts/{id}")
public String getPost(@PathVariable Long id) {
    Optional<Post> byId = postRepository.findById(id);
    Post post = byId.orElseGet(() -> new Post());
    return post.getTitle();
}

@GetMapping("/posts/{id}")
public String getPost(@PathVariable("id") Post post) {
    return post.getTitle();
}
```

DomainClassConverter가 파라미터로 id를 받아와 db에서 id로 찾은 post를 파라미터로 준다.

대신에 파라미터가 id가 아닌 post이기 때문에 @PathVariable("id")로 명시해줘야 한다.

##### Converter

어떤 하나의 Type을 다른 Type으로 변경해주는것이다.

DomainClassConverter는 자동으로 Converter registry에 들어간다. Spring MVC에서 어떤 데이터를 바인딩 받아서 컨버터할때 참고해서 사용한다. 크게 두가지 컨버터가 들어가 있는데 ToIdConverter와 ToEntityConverter가 들어가 있다.

*	ToEntityConverter : Id를 받아서 Entity로 변경 => Repository를 사용해서 찾아서 변경해준다.

* ToIdConverter : Entity를 받아서 Id로 변경

##### Formatter

String to Object or Object to String

id가 String이 아닐 수 있어 Formatter가 쓰일 수 없고 Converter가 쓰인것이다.

<br/>

## Pageable과 Sort

Spring MVC HandlerMethodArgumentResolver

* Spring MVC 핸들러 메소드의 매개변수로 받을 수 있는 객체를 확장하고 싶을 때 사용하는 인터페이스



pageable과 sort 관련 매개 변수

page : 0부터 시작

size : default => 20

sort : property,ASC|DESC (ex id,desc) default => asc
