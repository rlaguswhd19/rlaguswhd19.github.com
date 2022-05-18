---
title : "[JPA - SpringDataCommon] QueryDSL"
tags : 
 - Jpa
 - SpringDataCommon
---



# QueryDSL

1. 인터페이스에 Method를 하나씩 추가해야 하는 불편함.

2. Method 이름이 너무 길어서 해석이 오래걸림

   ex) findByFirstNameIngoreCaseAndLastNameStartsWithIgnoreCase(String firstName, String lastName) 



* Optional<T> findOne(**Predicate**): **이런 저런 조건**으로 무언가 하나를 찾는다.
* List<T>|Page<T>|.. findAll(**Predicate**): **이런 저런 조건**으로 무언가 여러개를 찾는다.
* [QuerydslPredicateExecutor 인터페이스](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/querydsl/QuerydslPredicateExecutor.html)



QueryDSL

* http://www.querydsl.com/타입 세이프한 쿼리 만들 수 있게 도와주는 라이브러리
* JPA, SQL, MongoDB, JDO, Lucene, Collection 지원
* [QueryDSL JPA 연동 가이드](http://www.querydsl.com/static/querydsl/4.1.3/reference/html_single/#jpa_integration)



Gradle 연동 방법

https://gaemi606.tistory.com/entry/Spring-Boot-Querydsl-%EC%B6%94%EA%B0%80-Gradle-7x
