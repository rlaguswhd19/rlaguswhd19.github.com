---
title : "[JPA - SpringDataCommon] CustumRepository 만들기"
tags : 
 - Jpa
 - SpringDataCommon

---



# CustumRepository 만들기

쿼리 메소드로 해결이 되지 않는 경우 직접 코딩으로 구현 가능

1. #### 기능 추가

   인터페이스를 하나 만들고 뒤에 Impl(규칙) 붙인 구현체 class를 하나 만든다.

   그리고 적용하려는 interface에 extends해주면 구현체의 함수가 실행된다.

   

2. #### 기본 기능 덮어쓰기

   기존의 함수를 똑같이 선언하고 구현체에서 Custom하여 덮어쓸 수 있다.

   <br/>

   ##### *MyRepository*

   ```java
   package com.hj.spring_jpa.jpa.custom;
   
   import java.util.List;
   
   public interface MyRepository<T> {
   
       // 추가 하기
       List<CustomPost> findMyPost();
   
       // 덮어 쓰기
       void delete(T entity);
   }
   
   ```

   ##### *MyRepositoryImpl*

   ```java
   package com.hj.spring_jpa.jpa.custom;
   
   import com.hj.spring_jpa.jpa.cascade.Post;
   import org.springframework.stereotype.Repository;
   import org.springframework.transaction.annotation.Transactional;
   
   import javax.persistence.EntityManager;
   import java.util.List;
   
   @Repository
   @Transactional
   public class MyRepositoryImpl implements MyRepository<CustomPost>{
   
       final EntityManager entityManager;
   
       public MyRepositoryImpl(EntityManager entityManager) {
           this.entityManager = entityManager;
       }
   
       @Override
       public List<CustomPost> findMyPost() {
           System.out.println("Custom findMyPost");
           return entityManager.createQuery("SELECT cp FROM CustomPost AS cp", CustomPost.class).getResultList();
       }
   
       @Override
       public void delete(CustomPost entity) {
           System.out.println("Custom delete");
           entityManager.remove(entity);
       }
   }
   ```

   <br/>

3. ##### 접미어 설정하기

   @EnableJpaRepositories(repositoryImplementationPostfix = "Erwin")로 Impl을 수정할 수 있다.

   ```java
   package com.hj.spring_jpa;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
   
   @SpringBootApplication
   @EnableJpaRepositories(repositoryImplementationPostfix = "Erwin")
   public class SpringJpaApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(SpringJpaApplication.class, args);
       }
   
   }
   ```

   

