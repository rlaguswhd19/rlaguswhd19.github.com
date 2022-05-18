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
@GetMapping("/posts/{id}")
    public String getPost(@PathVariable("id") Post post) {
        return post.getTitle();
    }

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



