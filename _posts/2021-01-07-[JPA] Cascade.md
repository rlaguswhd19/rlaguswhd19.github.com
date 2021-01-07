---
title: "[JPA] Cascade"
tags:
 -JPA
---



# Cascade

Entity의 상태변화를 전파시키는 옵션

Study가 변할때 Account의 Set도 Study의 상태를 전이시키고 싶을때는 Cascade 옵션을 사용해야 한다. 기본적으로 Cascade는 아무 옵션도 되어있지 않다.

<br/>

## Entity의 상태란?

<img src="https://user-images.githubusercontent.com/46040824/103860879-cc55f500-50ff-11eb-926e-18b1283b4596.JPG" width="100%"/>

* Transient: JPA가 모르는 상태

* Persistent: JPA가 관리중인 상태 (1차 캐시, Dirty Checking, Write Behind, ...)

  ```java
  @Component
  @Transactional
  public class JpaRunner implements ApplicationRunner{
  
  	@PersistenceContext
  	EntityManager entityManager;	
  	
  	@Override
  	public void run(ApplicationArguments args) throws Exception {
  		Account account = Account.builder()
  				.username("hj")
  				.password("pass")
  				.created(LocalDate.now())
  				.studys(new HashSet<>())
  				.build();
  		
  		Study study = Study.builder()
  				.name("JPA 공부를 해보자")
  				.build();
  
  		study.setOwner(account);
  		account.getStudys().add(study);
          
  		Session session = entityManager.unwrap(Session.class);
          
          // Persistent
  		session.save(account);
  		session.save(study);
          
          Account hj = session.load(Account.class, account.getId());
  		hj.setUsername("hj2");
       		hj.setUsername("hj3");
        		hj.setUsername("hj");
  		System.out.println("====================================");
  		System.out.println(hj.getUsername());
  	}
  }
  ```

  * 1차캐시

    여기서 session.save() 메소드를 실행했다고 해서 바로 Query가 발생하지 않는다. 이를 1차캐시라고 한다. 1차 캐시란 Session, EntityManger => Persistent Context라고 부르는데 Persistent Context에 인스턴트를 넣어논 상태다. 그래서 hj를 다시 부를때 Query가 발생하지 않고 Persistent Context에 있는것을 불러온다.

    위의 경우 print가 먼저 실행되고 후에 Insert Query가 발생하는것을 확인할 수 있다.

  * Dirty Checking(객체의 변경사항을 계속 감지한다), Write Behind(객체 상태 변화를 데이터베이스에 최대한 늦게 필요한 시점에 반영한다.)

    hj의 Username을 hj2로 변경했는데 자동으로 Update Query가 발생한다. 하지만 3번 변경해서 다시 hj로 돌아간다면 원래 hj였기 때문에 Update Query가 발생하지 않는다.

* Detached: JPA가 더 이상 관리하지 않는 상태

  * 트랜잭션이 끝나서 세션 밖으로 나왔을때

    Repository에서 객체를 return해 Service에서 받아서 쓰는 상태면 Session이 끝나기 때문에 더이상 관리하지 않는 상태

* Removed: JPA가 관리하긴 하지만 삭제하기로 한 상태

<br/>

## Cascade

도메인의 관계가 Parent와 Child관계에서 적용할 수 있다. Study와 Account는 그런 관계가 아니다.

Post와 Comment를 만들어보겠다.

<br/>

#### Post.class

```java
public class Post {
	
	@Id @GeneratedValue
	private Long id;
	
	private String title;
	
	@OneToMany(mappedBy = "post")
	private Set<Comment> comments = new HashSet<>();
	
	public void addComment(Comment comment) {
		this.getComments().add(comment);
		comment.setPost(this);
	}
	
}
```

<br/>

#### Comment.class

```java
public class Comment {
	
	@Id @GeneratedValue
	private Long id;
	
	private String comment;
	
	@ManyToOne
	private Post post;
}
```

이 관계에서는 Comment가 주인이다. 이제 Post를 저장해본다.

```java
@Component
@Transactional
public class JpaRunner implements ApplicationRunner{

	@PersistenceContext
	EntityManager entityManager;	
	
	@Override
	public void run(ApplicationArguments args) throws Exception {
		Post post = Post.builder()
				.title("JPA 공부해보기")
				.comments(new HashSet<>())
				.build();
		
		Comment comment1 = Comment.builder()
				.comment("언제 공부하지")
				.build();
		post.addComment(comment1);
		
		Comment comment2 = Comment.builder()
				.comment("지금부터라도 공부해")
				.build();
		post.addComment(comment2);
		
		session.save(post);
	}
}
```

이렇게 하면 Post는 저장되지만 comment1, comment2는 저장되지 않는다. 

Cascade를 설정해주면

```java
@OneToMany(mappedBy = "post", cascade = CascadeType.PERSIST)
private Set<Comment> comments = new HashSet<>();
```

Post만 저장해도 Comment가 저장된다.

<br/>

이상태에서 remove를 추가하고

```java
@OneToMany(mappedBy = "post", cascade = {CascadeType.PERSIST, CascadeType.REMOVE})
private Set<Comment> comments = new HashSet<>();
```

delete를 실행하면 Post가 삭제되면서 연관된 comment도 삭제된다.

```java
Session session = entityManager.unwrap(Session.class);
Post post = session.get(Post.class, 1L);
session.remove(post);
```