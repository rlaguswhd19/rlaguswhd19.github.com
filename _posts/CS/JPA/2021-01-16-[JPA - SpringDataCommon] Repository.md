---
title : "[JPA - SpringDataCommon] Repository"
tags :
 - Jpa
 - SpringDataCommon
---



# SpringDataCommon

![SpringData](https://user-images.githubusercontent.com/46040824/104797914-82f35d00-5805-11eb-94d4-36d62619c9b8.JPG)

## Repository

![SpringDataRepo](https://user-images.githubusercontent.com/46040824/104798045-aff43f80-5806-11eb-8782-a87e8b46fd04.JPG)

1. JpaRepository는 SpringbootJpa에서 제공해주는 인터페이스로 PagingAndSortingRepository를 상속받고 있는 인터페이스이다. 
2. PagingAndSorting은 SpringDataCommon에 속하는 인터페이스다. Paging과 Sorting을 제공하는 인터페이스로 CrudRepository를 상속 받고 있다. 
3. CrudRepository는 실질적인 여러 Queryt를 날릴 수 있는 기능들이 있다. CrudRepository에는 @NoRepositoryBean이 붙어있는데 Repository 인터페이스를 상속받았기 때문에 실제 Bean으로 등록하지 않도록 방지하기 위해 있는것이다.
4. Repository를 상속 받고 있는데 Marker 인터페이스로 실질적인 어떤 기능을 하는것은 아니다.

<br/>
이제 Junit을 활용해서 PostRepositoryTest를 작성해보겠다.

#### PostRepositoryTest.class

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class PostRepositoryTest {

	@Autowired
	PostRepository postRepository;
	
	@Before
	public void setUp() {
		postRepository.deleteAll();
	}
	
	@Test
	@Rollback(false)
	public void crudRepository() {
		Post post = Post.builder()
				.title("JPA 배우기")
				.comments(new HashSet<>())
				.build();
		
		assertThat(post.getComments()).isNotNull();
		assertThat(post.getId()).isNull();
		
		Post newPost = postRepository.save(post);
		assertThat(newPost.getId()).isNotNull();
		
		List<Post> posts = postRepository.findAll();
		assertThat(posts.size()).isEqualTo(1);
		assertThat(posts).contains(newPost);
	}
}
```

이걸 실행하면 insert query가 실행되지 않는다. 

* @DataJpaTest에 들어가보면 @Transactional이 붙어있고 기본적으로 모든 Test의 트랜잭션을 Rollback한다. Hibernate는 필요한 시점에만 데이터와 싱크하는데 어차피 Rollback할 것이기 때문에 Query를 실행시키지 않는것이다.
* 트랜잭션이 기본적으로 Rollback하는것은 Spring이 제공하는 기능이고 Query를 날리지 않은것은 JPA가 제공하는 기능이다. 
* newPost의 Id만 가져오고 Insert를 하지 않고 끝낸다. 그래서 @Rollback(false)를 붙여준다.

<br/>

이번엔 PagingAndSortRepository를 Test해보도록 하겠다.

```java
@Test
@Rollback(false)
public void pagingAndSortingRepository() {
	Post post = Post.builder()
			.title("아프리카 합격가즈아")
			.comments(new HashSet<>())
			.build();
	
	postRepository.save(post);
	
	Page<Post> pages = postRepository.findAll(PageRequest.of(0, 10));
	assertThat(pages.getTotalElements()).isEqualTo(1);
	assertThat(pages.getNumber()).isEqualTo(0);
	assertThat(pages.getSize()).isEqualTo(10);
	assertThat(pages.getNumberOfElements()).isEqualTo(1);
}
```

이렇게 하면 Page로 나오게 된다.

<br/>

PostRepository에 새로운 메소드를 우리가 코딩해서 만들 수도 있다.

```java
public interface PostRepository extends JpaRepository<Post, Long>{
	Page<Post> findByTitleContains(String title, Pageable pageable);
}
```

이렇게 만들면 메소드의 이름을 분석해서 Query를 만든다. 메소드에 대한 규칙은 나중에 더 자세히 다루도록 한다. Test를 실행해본다.

```java
@Test
@Rollback(false)
public void findByTitleContains() {
	for (int i = 0; i < 10; i++) {
		Post post = Post.builder()
				.title("JPA"+i)
				.comments(new HashSet<>())
				.build();
		
		postRepository.save(post);
	}
	
	Post post = new Post();
	post.setTitle("안녕");
	postRepository.save(post);
	
	Page<Post> posts = postRepository.findByTitleContains("녕", PageRequest.of(0, 10));
	System.out.println(posts.getContent());
}
```

```shell
Hibernate: 
    select
        post0_.id as id1_2_,
        post0_.title as title2_2_ 
    from
        post post0_ 
    where
        post0_.title like ? escape ? limit ?
2021-01-16 17:10:54.586 TRACE 15548 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [%안%]
2021-01-16 17:10:54.586 TRACE 15548 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [CHAR] - [\]
2021-01-16 17:10:54.588 TRACE 15548 --- [           main] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([id1_2_] : [BIGINT]) - [11]
[Post [id=11, title=안녕]]
```

