---
title: "[JPA] ORM Entity_Value 타입 맵핑"
tags:
 - Jpa
---



# Entity

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

<br/>

<br/>

# Value

## 엔티티 타입과 Value타입 구분

* 식별자가 있어야 하는가 (account의 경우 id)
* 독립적으로 존재해야 하는가

<br/>

## Value 타입 종류

* 기본 타입(String, Date, Boolean, ...)
* Composite Value 타입 => address의 경우
* Collection Value 타입
  * 기본 타입의 Collection
  * 컴포짓 타입의 Collection

<br/>

## Composite Value 맵핑

* @Embadable
* @Embadded
* @AttributeOverrides
* @AttributeOverride

<br/>

Account에 종속되어야 하는 데이터의 경우 Embadable과 Embadded를 사용하면 된다.

<br/>

#### Adderss.class

```java
@Embeddable
public class Address {

	private String street;
	
	private String city;
	
	private String state;
	
}
```

<br/>

#### Account.class

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
	
	private LocalDate created = LocalDate.now();
	
	@Embedded
	private Address address;
	
	@Transient
	private String trash;
}
```

<br/>

이런식으로 할 경우 Address는 테이블을 생성하는게 아닌 DB에서는 Account에 종속된 데이터가 되고 Java에서는 객체 상태로 활용할 수 있다.

#### 만들어진 Account Table

![account](https://user-images.githubusercontent.com/46040824/103847318-517ee100-50e3-11eb-9f41-e519646df095.JPG)

<br/>

Address를 여러 Value를 사용하고 싶은 경우는 @AttributeOverrides, @AttributeOverride를 사용하면 된다.

#### Account.class

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
	
	private LocalDate created = LocalDate.now();
	
	@Embedded
	@AttributeOverrides({
		@AttributeOverride(name = "street", column = @Column(name="home_street")),
		@AttributeOverride(name = "city", column = @Column(name="home_city")),
		@AttributeOverride(name = "state", column = @Column(name="home_state")),
	})
	private Address homeAdderss;
	
	@Embedded
	@AttributeOverrides({
		@AttributeOverride(name = "street", column = @Column(name="job_street")),
		@AttributeOverride(name = "city", column = @Column(name="job_city")),
		@AttributeOverride(name = "state", column = @Column(name="job_state")),
	})
	private Address jobAdderss;
	
	@Transient
	private String trash;
}
```

이런방식으로 각 속성들의 이름을 변경해주면 DB에서 Table을 만들어 준다.

<br/>

#### 만들어진 Account Table

![account2](https://user-images.githubusercontent.com/46040824/103847805-585a2380-50e4-11eb-8f55-d03c11c9d2cb.JPG)

