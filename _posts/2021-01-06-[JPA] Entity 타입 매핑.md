---
title: "[JPA] ORM Entity 타입 매핑"
tags:
 - JPA
---



# JPA Entity 타입 맵핑

Account라는 객체를 만들어 테이블로 맵핑하겠다.

lombok을 사용해서 간단하게 get, set 메소드와 생성 메소드를 만들었다.

```java
@Entity
@Builder
@Getter @Setter
@AllArgsConstructor @NoArgsConstructor
public class Account {
	
	@Id @GeneratedValue
	private Long id;
	
	@Column(unique = true)
	private String username;
	
	private String password;
	
	@Temporal(TemporalType.TIMESTAMP)
	private LocalDate created;
	
	@Transient
	private String trash;
}
```

<br/>

## @Entity

* 엔티티는 객체 세상에서 부르는 이름
* 보통 클래스와 같은 이름을 사용하기 때문에 값을 변경하지 않음
* 엔티티의 이름은 JQL에서 쓰임

@Entity(name = "myAccount")로 변경한다면 Hibernate 내부에서는 Enitity이름이 변경된다.

<br/>

## @Table

* 릴레이션 세상에서 부르는 이름
* @Entity의 이름이 기본값
* 테이블의 이름은 SQL에서 쓰임

실제 테이블에 맵핑되는 이름은 @Table(name = "")으로 변경할 수 있다.

<br/>

## @Id

* 엔티티의 주키를 맵핑할 때 사용
* 자바의 모든 primitive(long) 타입과 그 랩퍼(Long) 타입을 사용할 수 있음

<br/>

## @GeneratedValue

* 주키의 생성 방법을 맵핑하는 애노테이션
* 생성 전략과 생성기를 설정할 수 있다.

<br/>

## @Column

* unique
* nullable
* length
* columnDefinition
* ...

<br/>

## @Temporal

* 현재 JPA 2.1까지는 Date와 Calendar만 지원, LocalDateTime, LocalDate와 같은 타입은 2.2부터
* DATE = 날짜, TIME = 시간, TIMESTAMP = 날짜 + 시간

<br/>

## @Transient

* 컬럼으로 맵핑하고 싶지 않은 멤버 변수에 사용