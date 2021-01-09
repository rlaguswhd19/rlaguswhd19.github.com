---
title: "[Java] Stack_Heap"
tags: 
 - java
---



## Stack

1. Heap 영역에 생성된 Object 타입의 데이터의 참조값(주소값)이 할당된다.
2. 원시타입의 데이터가 값과 함께 할당된다.( 원시타입(primitive types) - byte, short, int, long, double, float, boolean, char )
3. 지역번수들은 scope에 따른 visibility를 가진다?
4. 각 Thread는 자신만의 Stack을 가진다.

Object타입의 데이터들에 대한 참조를 위한 값들이 할당된다.( Integer, String, ArrayList, ... )

함수 종료시 모든 지역변수들은 Stack에서 pop되어 사라진다.

Stack 메모리는 Thread 하나당 하나씩 할당된다. 각 Thread는 다른 Thread의 Stack영역 접근 불가

## Heap

```
1. Heap 영역에는 주로 긴 생명주기를 가지는 데이터들이 저장된다.(Object)
2. Object 타입( Integer, Character, Byte, Boolean, Long, Double, Float, Short, String)
3. 모두 Immutable이다. 힙에 저장될시 다른 함수에서 변경된다면 내부가 변경되는것이 아니라 새로운 오브젝트가 heap에 할당된다.
4.  Heap 영역에 있는 오브젝트들을 가리키는 레퍼런스 변수가 stack 에 올라가게 된다. 
5.  몇개의 스레드가 존재하든 상관없이 단 하나의 heap 영역만 존재한다. 
```

## Garbage Collection

```
public class Main {
    public static void main(String[] args) {
        String url = "https://";
        url += "rlaguswhd19.github.com";
        System.out.println(url);
    }
}
```

[![stack_heap](https://github.com/rlaguswhd19/CS/raw/master/asset/stack_heap.JPG)](https://github.com/rlaguswhd19/CS/blob/master/asset/stack_heap.JPG)

stack에 url이 올라가고 heap에 String https:// 가 올라간다

url += 가 끝나면 url은 heap에 새롭게 추가된 [https://rlaguswhd19.github.com이](https://rlaguswhd19.github.xn--com-5k0o/) 할당된다.

기존의 [https://는](https://xn--sh1b/) *Unreachable* 오브젝트가 된다.

JVM의 Garbage Collectior는 Unreachable Object를 우선적으로 메모리에서 제거하여 메모리 공간을 확보한다.

Unreachable Object 란 Stack 에서 도달할 수 없는 Heap 영역의 객체를 말한다.

Garbage Collection 과정은 Mark and Sweep 이라고도 한다.

JVM의 Garbage Collector 가 스택의 모든 변수를 스캔하면서 각각 어떤 오브젝트를 레퍼런스 하고 있는지 찾는과정이 Mark 다. Reachable 오브젝트가 레퍼런스하고 있는 오브젝트 또한 marking 한다.

첫번째 단계인 marking 작업을 위해 모든 스레드는 중단되는데 이를 stop the world 라고 부르기도 한다. (System.gc() 를 생각없이 호출하면 안되는 이유이기도 하다)

그리고 나서 mark 되어있지 않은 모든 오브젝트들을 힙에서 제거하는 과정이 Sweep 이다.

## JVM

C나 C++에서는 OS레벨의 메모리에 직접 접근하기 때문에 free()라는 메소드를 할당받았던 메모리를 명시적으로 해제해주어야 한다. 그렇지 않으면 memory leak이 발생하게 된다.

java는 OS의 메모리 영역에 직접적으로 접근하지 않고 JVM이라는 가상머신을 이용해서 간접적으로 접근한다. JVM은 C로 쓰여진 또 다른 프로그램인데, 오브젝트가 필요해지지 않는 시멎에서 알아서 free()를 수행하여 메모리를 확보한다. 메모리관리

OS에 요청한 사이즈 만큼의 메모리를 할당 받아서 실행하게된다. 할당 받은 이상의 메모리를 사용하게 되면 에러가 나면서 자동으로 프로그램이 종료되므로 실행한것만 죽고 다른것에 영향을 주지 않는다.