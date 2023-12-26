---
title : "[ISSUE] Elasticsearch rolling restart cluster down 이슈"
tags :
 - issue

---

# rolling restart

아프리카 방송국 ES 05, 06 서버의 스위치에 이슈가 발생하여 클러스터에서 제외해야하는 상황이 발생했다. 방송국 ES는 5.1.1 버전으로 되어있다.

방송국 ES의 각 노드 클러스터 설정이 04,05,06과 101,102,103에 설정이 모두 묶여있었다.

클러스터의 호스트 설정과 최소 마스터노드 갯수가 설정되어있다.

~~~yaml
discovery.zen.ping.unicast.hosts: []
discovery.zen.minimum_master_nodes = 7
~~~

마스터 노드는 8개가 있었고 최소 마스터노드가 7개로 되어있었다. 빼야하는 05,06은 마스터 노드가 포함되어 있어 최소 마스터 노드가 충족되지 않아 클러스터가 내려갔다. 처음에는 클러스터 호스트 설정으로 05,06과 통신하는 노드들이 해당 노드가 내려가 통신이 되지 않아 클러스터가 내려간줄 알았으나 이유는 마스터노드 갯수였다.

하지만 공식문서에서 클러스터 노드는 모든 노드들을 설정해주는게 좋다고 했기 때문에 해당 설정을 바꿔야 했고, 최소 [마스터 노드 갯수](https://esbook.kimjmin.net/03-cluster/3.3-master-and-data-nodes)는 `(전체 마스터 후보 노드 / 2) + 1`로 설정해야 네트워크 단절이 일어났을때 데이터 정합성을 맞출 수 있다.

최소 마스터 노드 설정은 7.x 부터는 자동으로 설정해주며 클러스터 호스트 설정은 **discovery.seed_hosts**로 변경되었다.

https://www.elastic.co/kr/blog/a-new-era-for-cluster-coordination-in-elasticsearch

<br/>

### discovery.zen.minimum_master_nodes

클러스터 설정을 확인해 보니 transient, persistent 설정이 있었다. 

###### 클러스터 설정 확인

~~~http
GET /_cluster/settings
~~~

~~~json
{
	"persistent": {},
	"transient": {}
}
~~~

transient > persistent > elasticsearch.yml으로 우선순위가 설정되어 있는데 transient는 휘발성 설정이고 persistent는 클러스터를 재시작 해도 남아있는 설정이다. transient에 최소 마스터 노드 설정이 되어있어 해당 설정을 바꾸고 노드를 내려줬다. 결과적으로 정상적으로 노드가 제외되었다.

<br/>

### cluster.routing.allocation.enable

이후 각 노드의 클러스터 설정을 모든 클러스터 노드와 통신하기 위해서 설정을 바꿔주고 각 노드들을 재시작 해야했다. 그러기 위해서 먼저 데이터 노드를 재시작할때 해당 노드들이 가지고 있는 샤드들이 재할당 되지 않게 하기 위해서 해당 설정을 바꿔줘야 했다. `none`을 써야 모든 샤드를 할당하지 않으나 공식 문서에서는 `primaries`를 추천한다. `primary` 샤드는 데이터 노드에 있어야 하고 없으면 `replica`를 `primary`로 승격한다.

~~~http
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": "primaries"
  }
}
~~~

그리고 elaticsearch.yml을 수정하고 노드를 재시작 해준다. 노드가 클러스터에 합류하면 해당 설정을 바꿔준다. 그러면 노드가 내려간 상태에서 해당 노드가 가지고 있던 샤드들을 다른 노드들에 할당하지 않는다. 그리고 내려갔던 노드가 다시 합류하면 샤드들을 그 노드에 그대로 할당한다. 다른 노드에 재할당할 경우 데이터를 옮기는 job을 수행하는데 오래 걸릴뿐더러 노드 자원을 많이 쓴다.

```http
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": null
    }
}
```

<br/>

### index.unassigned.node_left.delayed_timeout

unassigned shard가 있을때 NODE_LEFT(노드 제외)상태이 샤드일 경우 샤드를 할당하지 않고 기다리는 설정이다.

```http
PUT /_all/_settings
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m"
  }
}
```





