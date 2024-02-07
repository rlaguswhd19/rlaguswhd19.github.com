---
title : "[ElasticSearch] search after api"
tags :
 - ElasticSearch
---

# search after api

검색 페이징 처리할때 특정 시점(pit) 정렬된 데이터를 기준으로 데이터 페이징 처리 가능

~~~http
GET twitter/_search
{
    "query": {
        "match": {
            "title": "elasticsearch"
        }
    },
    "search_after": [1463538857, "654323"],
    "sort": [
        {"date": "asc"},
        {"tie_breaker_id": "asc"}
    ]
}
~~~



* [엘라스틱서치 공식 문서](https://www.elastic.co/guide/en/elasticsearch/reference/8.12/paginate-search-results.html)
* [point in time](https://www.elastic.co/guide/en/elasticsearch/reference/8.12/point-in-time-api.html)
* [블로그](https://velog.io/@nmrhtn7898/elasticsearch-%EA%B9%8A%EC%9D%80deep-%ED%8E%98%EC%9D%B4%EC%A7%80%EB%84%A4%EC%9D%B4%EC%85%98)



