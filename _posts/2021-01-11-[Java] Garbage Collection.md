---
title: "[Java] Garbage Collection"
tags :
 - Java
---

# Garbage Collection

메모리 누수를 막기 위해 사용하지 않는 객체의 메모리를 프로그래머가 직접 해제해야 했지만, Java에서는 JVM이 구성된  JRE가 제공되며, 그 구성 요소 중 하나인 Garbage Collection이 자동으로 사용하지 않는 객체를 해제한다.

<br/>

## stop-the-world

GC에 대해서 알아보기 전에 알아야 할 용어가 있다. 바로 'stop-the-world'이다. stop-the-world란, GC을 실행하기 위해 JVM이 애플리케이션 실행을 멈추는 것이다. stop-the-world가 발생하면 GC를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈춘다. GC 작업을 완료한 이후에야 중단했던 작업을 다시 시작한다. 어떤 GC 알고리즘을 사용하더라도 stop-the-world는 발생한다. 대개의 경우 GC 튜닝이란 이 stop-the-world 시간을 줄이는 것이다.

<br/>

## GC의 대상

* 객체가 null인 경우
* 블럭 실행 종료 후, 블럭 안에서 생성된 객체
* 부모 객체가 null인 경우, 포함하는 자식 객체

<br/>

## GC의 메모리 해제 과졍

1. ##### Marking

   ![mark](https://user-images.githubusercontent.com/46040824/104143145-ac754880-5401-11eb-8109-0e612a98286e.JPG)

   GC가 메모리를 사용하는 객체들을 Marking한다. 모든 오브젝트를 스캔하기 때문에 많은 시간을 소모한다.

   

2. ##### Normal Deletion

   ![deletaion](https://user-images.githubusercontent.com/46040824/104143143-ab441b80-5401-11eb-92ed-425ee7d6945a.JPG)

   Marking되어진 객체를 제외한 객체들을 제거하고 메모리를 반환한다.

   

3. ##### Compacting

   ![compacting](https://user-images.githubusercontent.com/46040824/104143144-ac754880-5401-11eb-9628-354823fae140.JPG)

   참조되지 않는 객체를 제거하고 남은 참조되어진 객체들을 묶어 메모리 공간을 확보한다. 이를 통해서 새로운 메모리 할당시 더 쉽고 빠르게 진행할 수 있다.

<br/>

## Weak Generational Hypothesis

GC는 `Weak Generational Hypothesis`에 기반한다.

* 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.
* 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.

이 가설의 장점을 최대한 살리기 위해 HotSpot VM에서 2가지의 물리적 공간을 나누었다. Young 영역과 Old 영역이다. 신규 생성되는 객체는 Young 영역에 보관하고, 오랫동안 살아남은 객체는 Old 영역에 보관한다.

<br/>

## Generational Garbage Collection



![garbage](https://user-images.githubusercontent.com/46040824/104143433-c2373d80-5402-11eb-95c9-c008b6a31ebf.JPG)

1. Young 영역(Yong Generation 영역)

   새롭게 생성한 객체의 대부분이 여기에 위치한다. 대부분의 객체가 금방 접근 불가능 상태가 되기 때문에 매우 많은 객체가 Young 영역에 생성되었다가 사라진다. 이 영역에서 객체가 사라질때 **Minor GC** 가 발생한다고 말한다.

2. Old 영역(Old Generation 영역)

   Young 영역에서 살아남은 객체가 여기로 복사된다. 대부분 Young 영역보다 크게 할당하며, 크기가 큰 만큼 Young 영역보다 GC는 적게 발생한다. 이 영역에서 객체가 사라질 때 **Major GC(혹은 Full GC)** 가 발생한다고 말한다.

3. Permanet 영역

   Method Area라고도 합니다. JVM이 클래스들과 메소드들을 설명하기 위해 필요한 메타데이터들을 포함하고 있다. JDK8부터는 PermGen은 Metaspace로 교체된다.

<br/>

## Generational Garbage Collection 과정

1. 새로운 객체를 Eden Space에 할당한다.

   ![1](https://user-images.githubusercontent.com/46040824/104143434-c3686a80-5402-11eb-9589-1dea65b26616.JPG)

2. Eden Space가 가득차게 되면 GC가 사용하지 않는 객체를 제거한다. 제거되지 않은 객체는 S0영역으로 이동된다.

   ![3](https://user-images.githubusercontent.com/46040824/104143436-c4010100-5402-11eb-9325-6c769c67025f.JPG)

3. 다음 minor GC일떄 S0와 Eden영역의 참조되지 않은 객체는 제거되고 S1으로 이동한다. 이때 살아남은 객체들은 aged가 1씩 증가한다. 그리고 S0는 clear상태가 된다.

   ![4](https://user-images.githubusercontent.com/46040824/104143437-c4010100-5402-11eb-9ed2-5fd4a7f5a610.JPG)

4. 이러한 minor GC과정이 반복되면서 살아남은 객체들은 aged가 증가하고 특정 기준을 넘게 되면 Old 영역으로 이동한다. (여기서는 aged > 8)

   ![5](https://user-images.githubusercontent.com/46040824/104143439-c4999780-5402-11eb-9729-f6ca67880da1.JPG)

   ![6](https://user-images.githubusercontent.com/46040824/104143440-c4999780-5402-11eb-9fc9-1625e4481604.JPG)



##### 참고자료

https://d2.naver.com/helloworld/1329

https://github.com/GimunLee/tech-refrigerator/blob/master/Language/JAVA/Garbage%20Collection.md#garbage-collection