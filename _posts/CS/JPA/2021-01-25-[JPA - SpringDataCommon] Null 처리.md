---
title : "[JPA - SpringDataCommon] Null 처리"
tags : 
 - JPA
 - SpringDataCommon
---



# Null 처리

스프링 데이터 2.0 부터 자바 8의 Optional을 지원

* Optional<Comment> findById(Long id);



Collection은 Null을 리턴하지 않고, 비어있는 Collection을 리턴합니다.

```java
	@Test
	public void OptionalTest() {
		Optional<Comment> byId = commentRepository.findById(100L);
		assertThat(byId).isEmpty();
		Comment comment = byId.orElseThrow(() -> new IllegalArgumentException());
		
		List<Comment> comments = commentRepository.findAll();
        if(comments != null){
            // 필요 없는 코드 비어있는것으로 나온다.
        }
		assertThat(comments.isEmpty());
	}
```



<br/>

스프링 프레임워크 5.0부터 지원하는 Null 애노테이션 지원

* @NonNullApi, @NonNull, @Nullable
* 런타임 체크 지원
* JSR 305 애노테이션 메타 애노테이션으로 가지고 있음. (IDE 및 빌드 툴 지원)

#### MyRepository.interface

```java
@NoRepositoryBean
public interface MyRepository<T, Id extends Serializable> extends Repository<T, Id>{
	<E extends T> E save(@NonNull E entity);
	List<T> findAll();
	long count();
	
	@Nullable //리턴 값
	<E extends T> E findById(@Nullable Id id); //파라미터 값
	
//	<E extends T> Optional<E> findById(Id id);
}
```

메소드 위에 @Nullable를 설정하면 리턴값에 설정되고 파라미터에 하고싶을땐 파라미터 앞에 하면 된다.





