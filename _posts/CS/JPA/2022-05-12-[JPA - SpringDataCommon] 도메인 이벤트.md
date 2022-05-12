---
title : "[JPA - SpringDataCommon] 도메인 이벤트"
tags : 
 - Jpa
 - SpringDataCommon
---



# 도메인 이벤트

도메인의 변화 @Entity라고 정의해둔 도메인 클래스의 변화를 이벤트로 발생시키는것 그리고 이벤트를 리스닝하는 이벤트 리스너가 도메인 변화를 감지하고 이벤트 기반의 프로그래밍이 가능

도메인 이벤트를 만들고 이벤트를 퍼블리싱하고 리스너하는게 스프링에는 기능이 내재되어 있다.

왜냐면 ApplicationContext가 이벤트 퍼블리셔다.

`ApplicationContext extends ApplicationEventPublisher`

<br/>

* Event

  도메인 이벤트 객체이다.

  * extends ApplicationEvent

    ```java
    public class CustomPostPublishedEvent extends ApplicationEvent {
        private final CustomPost customPost;
    
        public CustomPostPublishedEvent(Object source) {
            super(source);
            this.customPost = (CustomPost) source;
        }
    }
    ```

<br/>

* Listener

  이벤트가 발생했을때 어떤 작업을 할건지 프로그래밍 할 수 있다.

  * implements ApplicationListener<E extends ApplicationEvent>

    ```java
    public class CustomPostListener implements ApplicationListener<CustomPostPublishedEvent> {
    
        @Override
        public void onApplicationEvent(CustomPostPublishedEvent event) {
            System.out.println("========================");
            System.out.println(event.getCustomPost().getTitle() + " is published");
            System.out.println("========================");
        }
    }
    ```

    

  * @EventListener

    ```java
    public class CustomPostListener {
    
        @EventListener
        public void onApplicationEvent(CustomPostPublishedEvent event) {
            System.out.println("========================");
            System.out.println(event.getCustomPost().getTitle() + " is published");
            System.out.println("========================");
        }
    }
    ```

<br/>

스프링 데이터의 도메인 이벤트 Publisher

* @DomainEvents
* @AfterDomainEventPublication
* extends AbstractAggregationRoot<E>
* 현재는 save() 할 때만 발생한다.

도메인에 extends AbstractAggregateRoot<CustomPost>를 해준다. 그리고 save할때 이벤트를 등록하여 Listener가 동작하게 한다.

##### CustomPost

```java
public class CustomPost extends AbstractAggregateRoot<CustomPost> {
    
    public CustomPost publish() {
        this.registerEvent(new CustomPostPublishedEvent(this)); // 이벤트 등록
        return this;
    }
}
```



##### CustomPostRepositoryTest

```java
public void crud() {
        CustomPost customPost = new CustomPost();
        customPost.setTitle("Test");

        assertThat(customPostRepository.contains(customPost)).isFalse();

        customPostRepository.save(customPost.publish()); // 이순간 이벤트를 등록하고 Listener가 동작함

        assertThat(customPostRepository.contains(customPost)).isTrue();

        customPostRepository.delete(customPost);
        customPostRepository.flush();

    }
```

<br/>

##### AbstractAggregationRoot

* registerEvent
* (@AfterDomainEventPublication)clearDomainEvents 이벤트 비우는곳
* (@DomainEvents)domainEvents 이벤트 모아놓는곳

현재 save() 할때 발생

save할때 자동으로 AbstractAggregationRoot를 통해 domainEvents에 모아져있던 이벤트를 다 발생시키고 만들어 두었던 EventListener가 동작한다.
