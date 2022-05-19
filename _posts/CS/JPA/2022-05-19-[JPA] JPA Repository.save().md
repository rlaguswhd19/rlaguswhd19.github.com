---
title : "[JPA] JPA Repository.save()"
tags : 
 - Jpa
---



## save()

JpaRepository.save()는 단순히 엔티티를 추가하는 메소드가 아니다. 

* Transient 상태의 객체라면 persist()
* Detached 상태의 객체라면 merge()



##### Transient or Detached 판단하는법

[JPA Entity] (https://rlaguswhd19.github.io/2021/01/07/JPA-Cascade.html)

* 엔티티의 @Id 프로퍼티를 찾는다. 해당 프로퍼티가 null이면 Transient 상태로 판단하고 id가 null이 아니면 Detached 상태로 판단한다.
* 엔티티가 Persistable 인터페이스를 구현하고 있다면 isNew() 메소드에 위임한다.
* JpaRepositoryFactory를 상속받는 클래스를 만들고 getEntityInfomation()을 오버라이딩해서 자신이 원하는 판단 로직을 구현할 수도 있다.



##### EntityManager.persist()

* https://docs.oracle.com/javaee/6/api/javax/persistence/EntityManager.html#persist(java.lang.Object)
* Persist() 메소드에 넘긴 그 엔티티 객체를 Persistent 상태로 변경합니다.

![image](https://user-images.githubusercontent.com/46040824/168997335-8ec59f72-ddf2-436f-9779-eaf18266810f.png)



##### EntityManager.merge()

* https://docs.oracle.com/javaee/6/api/javax/persistence/EntityManager.html#merge(java.lang.Object)
* Merge() 메소드에 넘긴 그 엔티티의 복사본을 만들고, 그 복사본을 다시 Persistent 상태로 변경하고 그 복사본을 반환합니다.

![image](https://user-images.githubusercontent.com/46040824/168997377-c99c60ff-cf23-498a-8397-e5617a471a57.png)

