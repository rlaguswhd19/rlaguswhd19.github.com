---
title : "[monbodb] init"
tags : mongodb
---

# Mongodb

<br/>

### Access Pattern

Application이 데이터에 접근하는 패턴을 파악하여 collection을 정의한다.

* *함께 조회되는 경우가 빈번한 데이터들은 같은 collection에 담아, query의 횟수를 줄일 수 있다.*

- *주로 읽기만 하는 데이터와 자주 업데이트하는 데이터는 별개의 collection에 담는다.*



### Relation

이커머스 플랫폼을 운영하는 입장에서 Product와 Category라는 두 collection이 db에 존재한다고 할떄 rdb에서는 category_id라는 column으로 Category table을 join하여 가져오도록 설계한다.

![rdb](https://github.com/rlaguswhd19/rlaguswhd19.github.com/assets/46040824/a4feb88e-6322-47b8-8373-615477069596)

<br/>

이런 entity간의 relation을 mongodb에서는 reference, embed 중에 결정해야 한다.

##### reference

collection간 참조할 수 있도록 id를 저장하는 것

![reference](https://github.com/rlaguswhd19/rlaguswhd19.github.com/assets/46040824/483be928-cbec-4347-a1ae-bcbdf1b7a448)

<br/>

##### embed

관계된 document를 함께 저장하는 것

![embed](https://github.com/rlaguswhd19/rlaguswhd19.github.com/assets/46040824/0a569f6d-eb45-47d9-90dc-48e67dfbe5ef)



application의 성격에 따라 기준을 가지고 고민을 해야한다. 함께 읽는거라면 `embed`를 사용한다 하지만 자주 변환되는 값이면 `reference`를 사용한다. 예를 들어 카테고리 정보가 끊임없이 변경되는 상황이라면 어떨까? Embed 방식의 경우, 해당 카테고리의 모든 상품 document를 찾아서 일일이 embed 된 카테고리 정보를 수정해야 한다. 반면, reference 방식의 경우 별도로 관리되는 카테고리 collection에서 하나의 document만 찾아 수정하면 된다.

**reference는 데이터를 정규화하고 embed는 데이터를 비정규화 한다**.

일반적으로 reference(정규화)는 쓰기를 빠르게 하고, embed(비정규화)는 읽기를 빠르게 한다. 분당 수천~수만 번씩 access되는 웹페이지에서 이용되는 정보라면 하나의 document에 모아두는게 필요하다.

|           | **장점**                                       | **단점**                                      |
| --------- | ---------------------------------------------- | --------------------------------------------- |
| Embed     | 읽기가 빠르고, 효율적(query 한 번에 모두 조회) | 수정이 비효율적(여러 document 중복 작업 필요) |
| Reference | 쓰기가 빠르고, 효율적(document 하나만 수정)    | 읽기가 비효율적(query를 여러번 실행)          |

<br/>

### 스키마 설계 Example

온라인 서점을 운영하는 입장에서 스키마 설계 예제를 진행한다.

하나의 책에 여러 저자가 참여할 수 있고 한 저자가 여러 책을 집필할 수 있기에 책(Book)과 저자(Author)는 Many-to-Many의 관계를 갖는다. rdb의 경우 Book과 Author 사이에 BookAuthor라는 mapping table을 두어 책과 저자의 다대다 관계를 두 개의 일대다 관계로 설계한다.

![rdb설계](https://github.com/rlaguswhd19/rlaguswhd19.github.com/assets/46040824/94893b7d-3c3a-4a65-9d43-dcf25c181828)

RDB의 설계 철학을 그대로 MongoDB에 적용하였다. 각각의 속성(필드)은 하나의 엔티티(컬렉션)에 속해있고, 정규화를 통해 중복되는 데이터는 없다. 사실 스키마 자체는 크게 문제 될 요소는 없다. 예를 들어, 데이터의 일관성이 중요하고 속도에 크게 신경 쓸 필요가 없다면 이 스키마 그대로 사용해도 상관없다. 오히려, 책과 저자 정보가 실시간으로 계속 업데이트되고 있다면 이렇게 정규화한 RDB 스러운 스키마가 더 바람직하다고 할 수 있다.

하지만 장사가 너무 잘되어 매분마다 수만 번의 db 조회가 발생한다면 embedded model이 적합할 수 있다. 위와 같은 모델은 1) Book Collection 조회, 2) Book ID로 BookAuthor Collection 조회, 3) Author ID로 Author Collection 조회, 총 3번의 query를 수행해야 하는 치명적인 단점이 존재한다. 만약 author 정보를 Book document에 embed 시켜놓았으면, query 한 번으로 끝날 작업을 3 단계로 나누어서 하게 됨으로써, DB 서버에 과부하를 줄 수도 있고 페이지 로딩 속도가 느려져 사용자 경험에 악영향을 준다.

![mongo설계](https://github.com/rlaguswhd19/rlaguswhd19.github.com/assets/46040824/79312056-59c4-4535-9b60-0eb78cd21750)

그런 이유로, BookAuthor라는 중간 collection을 없애고, author 정보를 Book document에 embed 했다. 이제 책과 저자 정보를 query 한 번에 가져올 수 있어서 읽기 성능이 재고될 것으로 기대된다. 그럼 모든 문제가 해결된 것일까요? 조금 더 현실에 가까운 예제를 위해, 사용자들이 관심 작가를 follow 할 수 있는 기능을 추가해본다.

![follower 추가](https://github.com/rlaguswhd19/rlaguswhd19.github.com/assets/46040824/79f911a2-a14f-4057-aa8a-0099ff26a4fd)

 Embedded 모델에 follower 정보를 추가하면 Author 아래에 follower라는 array가 추가되는 형태일 것이다. 그럼 이제 한 가지 문제가 생기는데 한강 작가에게 follower가 추가될 때마다 작가의 모든 저서의 document를 순회하면서 authors array에서 한강 작가에 해당하는 document를 찾고, 이 document의 followers array에 신규 follower 정보를 추가해야 한다. Follower가 following을 취소할 때 역시 마찬가지로 이런 비효율적인 과정을 통해 follower 정보를 삭제해야 한다. 이는 Book document에 Author의 모든 속성을 embed 하는 것이 비효율적임을 보여준다.

![hibrid model](https://github.com/rlaguswhd19/rlaguswhd19.github.com/assets/46040824/69a598f3-45bd-4d3d-aecc-f90a5765de49)

이는 Author document 필드 각각의 성격을 고려하지 않고 일괄적으로 embed 하여 발생하는 문제이다. 어떤 필드는 거의 읽기만 수행하는 정적인 값이고, 어떤 필드는 자주 수정이 발생하는 동적인 성격을 갖는다면 이 두 필드는 별개로 처리되어야 한다. 지금 예제에서 작가의 이름은 정적인 값이고, follower는 끊임없이 수정이 발생하는 동적인 정보다. 이런 경우 정적 필드는 embed 시켜 함께 조회되도록 하고, 동적인 필드는 개별 collection에 저장하여 효율적으로 수정될 수 있도록 하는 `hybrid` model이 권장한다.

 이미 많이 최적화하였지만, follower 정보를 추가한 김에 하나의 상황을 더 다루어본다. 예시로 들던 교보문고 홈페이지를 보면 한강, 김영하 작가처럼 소수의 인기 작가들은 수천 명의 follower가 있지만, 대다수의 경우 없거나 한 두 명인 것을 확인할 수 있다. 그렇다고 이런 소수의 인기 작가의 followers array에 무한정 데이터를 추가할 수는 없다. MongoDB의 물리적인 한계(doc size < 16Mb)도 있고, 한 번의 조회에 너무 많은 정보를 읽어오면서 application에 무리가 생길 수도 있다.

![flag 추가](https://github.com/rlaguswhd19/rlaguswhd19.github.com/assets/46040824/680f0dda-2049-40b3-ac92-c6a7147b4c2a)

 이런 경우, 해당 document가 outlier임을 나타내는 flag를 저장하고(위 예시의 "tbc"), Id를 기준으로 여러 document에 나누어 저장하는 방식으로 설계한다. 이제 application은 flag가 존재하는 경우에만 추가 query를 수행하여 follower 정보를 가져오게 됩니다. 이로써, 우리의 북스토어 서버는 **필요한 상황에 필요한 만큼만 데이터를 가져오도록** 최적의 스키마를 갖추었다.

### 권장

| **Embed**                          | **Reference**                        |
| ---------------------------------- | ------------------------------------ |
| 변경이 (거의)없는 정적인 데이터    | 변경이 잦은 데이터                   |
| 함께 조회되는 경우가 빈번한 데이터 | 조회되는 경우가 많지 않은 데이터     |
| 빠른 읽기가 필요한 경우            | 빠른 쓰기가 필요한 경우              |
| 결과적인 일관성이 허용될 때        | 즉각적으로 일관성이 충족되어야 할 때 |



*출처*

https://dev.gmarket.com/32