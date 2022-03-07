---
title : "[SPRINGBOOT] RestTemplate vs WebClient"
tags :
 - Springboot
---

## Blocking / Non-blocking

행위자가 취한 행위 자체가, 또는 그 행위로 인해 다른 무엇이 **막혀버린**, **제한된**, **대기하는** 상태.
대개의 경우에는 나 이외의 대상으로 하여금 내가 Block 당하겠지만(Blocked), 어찌 되었든 문자 자체로는 나라는 **단일 개체 스스로의 상태**를 나타낸다.

- `호출된 함수`가 자신이 할 일을 모두 마칠 때까지 제어권을 계속 가지고서 `호출한 함수`에게 바로 돌려주지 않으면 Block
- `호출된 함수`가 자신이 할 일을 채 마치지 않았더라도 바로 제어권을 건네주어(return) `호출한 함수`가 다른 일을 진행할 수 있도록 해주면 Non-block

<br/>

## Synchronous / Asynchronous

**동시에 발생하는** 것들(always plural, can never be singular).
동시라는 것은 즉, 시(time)라는 단일계(system)에서 **같이**, **함께** 무언가가 이루어지는 두 개 이상의 개체 혹은 이벤트를 의미한다고 볼 수 있다.

- `호출된 함수`의 수행 결과 및 종료를 `호출한 함수`가(`호출된 함수`뿐 아니라 `호출한 함수`도 함께) 신경 쓰면 Synchronous
- `호출된 함수`의 수행 결과 및 종료를 `호출된 함수` 혼자 직접 신경 쓰고 처리한다면(as a callback fn.) Asynchronous

<br/>

## Request

- RestTemplate
  - Spring 3부터 지원, REST API 호출이후 응답을 받을 때까지 기다리는 동기 방식.
- AsyncRestTemplate
  - Spring 4에 추가된 비동기 RestTemplate이다.
- WebClient
  - Spring 5에 추가된 논블럭, 리엑티브 웹 클라이언트로 동기, 비동기 방식을 지원한다.

<br/>

## RestTemplate

- spring 3.0 부터 지원한다.
- 스프링에서 제공하는 http 통신에 유용하게 쓸 수 있는 템플릿.
- Multi-Thread와 Blocking방식을 사용.

![restTemplate 방식](https://user-images.githubusercontent.com/46040824/156978869-df9fc21e-eb62-4d2e-8f56-48b3f89c27c8.png)

<br/>



## WebClient

* Spring 5.0에서 추가된 인터페이스

* Single-Thread와 Non-Blocking 방식을 사용

  



![WebClient 방식](https://user-images.githubusercontent.com/46040824/156979080-9ed31f7b-5c0d-4aaa-9b25-2a2bc9927bbc.jpg)



## 성능 비교

![성능비교](https://user-images.githubusercontent.com/46040824/156979174-f0ebbff7-f122-4432-b9e9-f9c47e7c0d14.png)



##### 출처

* https://musma.github.io/2019/04/17/blocking-and-synchronous.html - 동기 / 비동기, 블럭 / 논블럭

* https://happycloud-lee.tistory.com/220 - RestTemplate과 WebClient 차이점