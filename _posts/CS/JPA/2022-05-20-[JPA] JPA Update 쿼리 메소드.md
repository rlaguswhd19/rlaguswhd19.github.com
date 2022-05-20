---
title : "[JPA] Update 쿼리 메소드"
tags : 
 - Jpa
---



# Update 쿼리 메소드

##### 쿼리 생성하기

* find
* count
* delete
* update?

주로 update는 PersistContext가 관리를 하다가 객체 상태의 변화가 일어나고 데이터 베이스에 동기화를 해야겠다 => flush() 하면 update 쿼리가 날아간다.



##### Update 또는 Delete 쿼리 직접 정의하기 (추천 x)

* @Modifying @Query

```java
@Modifying
@Query("UPDATE Post p SET p.title = ?1 WHERE p.id = ?2")
int updateTitle(String hibernate, Long id);
```



##### PostRepositoryTest

```java
@Test
public void updateTest() {
    Post post = savePost();
    int updated = postRepository.updateTitle("hibernate", post.getId());

    assertThat(updated).isEqualTo(1);

    Optional<Post> byId = postRepository.findById(post.getId());
    assertThat(byId.get().getTitle()).isEqualTo("test"); // hibernate가 아니다.
}
```

select쿼리가 안날라간다. persistContext에서 관리하고 있기 때문에 한 Transaction내에서는 캐시가 계속 유지가 된다. 캐시가 비워지지 않았기 때문에 계속 persist 상태의 객체를 find를 하면 db에 쿼리를 날리지 않고 캐싱하고 있던 객체를 가져온다.

<br/>

그래서 Modifying에는 `flushAutomatically`와 `clearAutomatically`가 있다.

```java
@Modifying(clearAutomatically = true)
@Query("UPDATE Post p SET p.title = ?1 WHERE p.id = ?2")
int updateTitle(String hibernate, Long id);
```



##### @Modifying

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.METHOD, ElementType.ANNOTATION_TYPE })
@Documented
public @interface Modifying {

	/**
	 * Defines whether we should flush the underlying persistence context before executing the modifying query.
	 *
	 * @return
	 */
	boolean flushAutomatically() default false;

	/**
	 * Defines whether we should clear the underlying persistence context after executing the modifying query.
	 *
	 * @return
	 */
	boolean clearAutomatically() default false;
}
```

그래서 별로 추천하지 않는 방법이다.

<br/>

```java
@Test
public void updateTest2() {
    Post post = savePost();
    post.setTitle("hibernate");

    List<Post> all = postRepository.findAll();
    assertThat(all.get(0).getTitle()).isEqualTo("hibernate");
}
```

명시적으로 update를 하지 않았지만 find하기전에  db의 싱크를 맞춰야 한다고 hibernate가 알고 있기 때문에 데이터 변경을 반영해준다.

flush와 clear를 모든 팀원이 이해해야지만 위의것을 이해할 수 있다.
