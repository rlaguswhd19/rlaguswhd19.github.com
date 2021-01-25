---
title : "[JPA - SpringDataCommon] Interface 정의"
tags : 
 - JPA
 - SpringDataCommon
---



# Interface 정의

JpaRepository를 통해서 많은 기능들이 들어오는데 그것이 싫고 나는 내가 정의한 기능들을 사용하고 싶을 때 Interface를 정의하여 사용한다.

<br/>

#### CommentRepository.interface

```java
@RepositoryDefinition(domainClass = Comment.class, idClass = Long.class)
public interface CommentRepository {
	Comment save(Comment comment);
	List<Comment> findAll();
}
```

이런 방식으로 내가 만들어서 사용한다. 이를 확인하기 위해 Test를 작성해본다.



#### CommentRepositoryTest.java

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class CommentRepositoryTest {

	@Autowired
	CommentRepository commentRepository;
	
	@Test
	public void crud() {
		Comment comment = Comment.builder()
				.comment("합격할 수 있을까")
				.build();
		commentRepository.save(comment);
		
		List<Comment> list = commentRepository.findAll();
		assertThat(list.size()).isEqualTo(1);
	}
}
```

간단하게 정의한 메소드가 동작하는지 확인했다.

그럼 내가 Interface를 정의하여 사용할 때  모든 Interface에 기본적으로 save와 findAll을 사용하고 싶을 때는 어떻게 해야하나..?

<br/>

#### MyRepository.class

```java
@NoRepositoryBean
public interface MyRepository<T, Id extends Serializable> extends Repository<T, Id>{
	<E extends T> E save(E entity);
	List<T> findAll();
}

```

이렇게 Entity를 받아서 사용하는것으로 한다. 그리고 이를 상속하는 Interface를 만든다.

```java
public interface CommentRepository extends MyRepository<Comment, Long>{}
```

MyRepository를 상속받아 사용하면 기본적으로 save와 findAll메소드를 사용할 수 있다.