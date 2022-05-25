---
title : "[JPA] Sort"
tags : 
 - Jpa
---



# Sort

Pageable이나 Sort를 매개변수로 사용할 수 있는데  @Query와 같이 사용할 때 제약 사항이 있다.

Order by 절에서 함수를 호출하는 경우에는 Sort를 사용하지 못하고 JpaSort.unsafe()를 사용해야 한다.

* Sort는 그 안에서 사용한 프로퍼티 또는 Alias가 엔티티에 없는 경우에는 예외가 발생한다.

* JpaSort.unsafe()를 사용하면 함수 호출을 할 수 있다.

  ```java
  postRepository.sortTest("title", JpaSort.unsafe(Sort.Direction.DESC, "LENGTH(title)"));
  ```

  
