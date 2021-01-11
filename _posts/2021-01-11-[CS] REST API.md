---
title: "[CS] REST API"
tags : 
 - cs
---



# REST API

### API

* Application Programming Interface

### REST

* REpresentational State Transfer
* 인터넷 상의 시스템간의 상호 운용성을 제공하는 방법중 하나
* 시스템 제각각의 독립적인 진화를 보장하기 위한 방법
* REST API: REST 아키텍쳐 스타일을 따르는 API

#### REST 아키텍쳐 스타일

* Client-Server

* Stateless

* Cache

* **Uniform Interface**

  * Identification of resources

    리소스가 URI로 식별되며 됀다.

  * manipulation of resources through representations

    리소스를 만들거나 업데이트하거나 삭제하거나 이럴때 HTTP 메세지에 담아서 전송해야한다.

  * **self-descriptive messages**

  * **hypermedia as the engine of application state(HATEOAS)**

* Layered System

* Code-On-Demand(optional)

  * 서버에서 코드를 클라이언트로 보내서 실행할 수 있어야 한다 -> javascript

<br/>

## self-descriptive message

*	메세지는 스스로를 설명해야한다.
*	확장 가능한 커뮤니케이션
  *	서버나 클라이언트가 변경되더라도 오고가는 메세지는 언제나 self-descriptive 하므로 해석이 가능하다.

<br/>

Content-Type이 없어서 어떤 형식으로 작성된 응답인지 알 수 가 없다. 그래서 Content-Type 헤더를 추가해줬다.

![self1](https://user-images.githubusercontent.com/46040824/104153388-98434280-5425-11eb-8f1d-2346d36e5ee6.JPG)![self](https://user-images.githubusercontent.com/46040824/104153389-99746f80-5425-11eb-973d-5e65d841df3d.JPG)

그러나 이래도 op가 무엇인지 path가 무엇인지에 대한 정보가 없기 때문에 아직 self-descriptive하지 못하다.
<br/>

![self2](https://user-images.githubusercontent.com/46040824/104153618-25869700-5426-11eb-8a69-87750078ff7d.JPG)

이렇게 하면 완전히 self-descriptive하다. json-patch+json이란 미디어타입의 명세가 있기 때문에 명세를 찾아서 해석을 하면 된다.

<br/>

## HATEOAS

* 애플리케이션의 상태는 Hyperlink를 이용해서 전이되어야 한다.
* 애플리에키션 상태 전이의 late binding
  * 어디서 어디로 전이가 가능한지 미리 결정되지 않는다. 어떤 상태로 전이가 완료되고 나서야 그 다음 전이될 수 있는 상태가 결정된다. 쉽게 말해서 링크는 동적으로 변경될 수 있다.

<br/>

![hate](C:\Users\rlagu\OneDrive\바탕 화면\hate.JPG)

이것을 보면 Link정보가 있어서 전 게식물에 대한 URI와 다음 게시물에 대한 URI를 응답에 포함하여 다음 상태로 전이할 수 있다.

<br/>

## 왜 **Uniform Interface**를 해야하나

독립적 진화

* 서버와 클라이언트가 각각 독립적으로 진화한다.
* **서버의 기능이 변경되어도 클라이언트가 업데이트할 필요가 없다.**
* REST를 만들게 된 계기: How do I improve HTTP without breaking the web

<br/>

# self-descriptive와 Hateoas를 만족하는 방법

#### self-descriptive

*	custom media type => IANA에 미디어 타입을 등록하고 그 미디어 타입을 리소스로 리턴할 때 Content-Type으로 사용한다.
*	profile link relation => Docs를 만들어서 link에 담아 해석

<br/>

#### HATEOAS

* http 헤더나 본문에 링크를 담아 만족시킬 수 있다.

<br/>

##### 참고자료

* https://www.youtube.com/watch?v=RP_f5dMoHFc
* 백기선 인프런 REST API 강의

