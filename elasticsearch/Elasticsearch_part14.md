# 클러스터 성능 모니터링과 최적화

- Elasticsearch는 자체적으로 monitoring 기능을 지원한다.

  - Kibana에서 monitoring을 사용하면 자동으로 monitoring 데이터가 쌓이는데 이는 ES 내부의 monitoring 기능을 사용하는 것이다.
  - 아래와 같은 것들을 모니터링 할 수 있다.
    - cluster
    - node
    - index
  - 기본적으로 `_cat` API로 요청을 보냈을 때 응답으로 오는 데이터들이 포함되어 있다.
    - 다만, 모두 포함 된 것은 아니다.
  - 한 클러스터 뿐 아니라 다른 클러스터에서 monitoring 데이터를 가져오는 것도 가능하다.

  - `_cat` API에 대응되는 API들
    - `_cat` API는 대부분 사람이 command line이나 Kibana console을 통해 직접 사용하는 것을 의도하고 만들어졌다.
    - 따라서 application에서 사용하기는 적절하지 않다.
    - Application에서 사용해야 한다면 `_cat`에 대응되는 API를 사용하는 것을 권장한다.

  | _cat          | application용 API                                            |
  | ------------- | ------------------------------------------------------------ |
  | _cat/health   | [Cluster health API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-health.html) |
  | _cat/indicise | [Get index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-get-index.html#indices-get-index) |
  | _cat/nodes    | [Nodes info API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-info.html) |
  | _cat/segments | [Index segments API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-segments.html) |



- monitoring data 저장하기

  - 아래와 같이 클러스터 설정만 변경해주면 된다.

  > https://www.elastic.co/guide/en/elasticsearch/reference/current/monitoring-settings.html
  >
  > https://www.elastic.co/guide/en/elasticsearch/reference/current/collecting-monitoring-data.html

  ```bash
  PUT _cluster/settings
  {
    "persistent": {
      "xpack.monitoring.collection.enabled": true
    }
  }
  ```



- 자체적으로 지원하는 모니터링 기능 외에도 metricbeat, filebeat 등을 사용 가능하다.





## 클러스터의 상태 확인하기

- `_cat/health`

  - 클러스터의 상태를 확인하는 API
  - 기본형

  ```http
  GET _cat/health
  ```

  - `v` 옵션을 추가
    - 결과 값에 header를 추가해서 보여준다.

  ```http
  GET _cat/health?v
  ```

  - format 옵션을 추가
    - 지정한 format으로 보여준다.

  ```http
  GET _cat/health?format=json
  ```



- 클러스터 상태 정보

  - `format=json` 옵션을 줬을 때의 응답

  ```json
  [
    {
      "epoch" : "1620718739",		// API를 호출한 시간을 UNIX 시간 형태로 표현한 숫자
      "timestamp" : "07:38:59",	// API를 호출한 시간
      "cluster" : "es-docker-cluster",	// 클러스터 이름
      "status" : "green",		// 클러스터 상태
      "node.total" : "4",		// 전체 노드의 개수
      "node.data" : "3",		// 데이터 노드의 개수
      "shards" : "6",			// 샤드의 개수
      "pri" : "3",			// 프라이머리 샤드의 개수
      "relo" : "0",			// 클러스터에서 재배치되고 있는 샤드의 개수
      "init" : "0",			// 초기화되고 있는 샤드의 개수
      "unassign" : "0",		// 배치되지 않은 샤드의 개수
      "pending_tasks" : "0",	// 클러스터의 유지, 보수를 위한 작업 중 실행되지 못하고 큐에 쌓여 있는 작업의 개수
      "max_task_wait_time" : "-",	// pending_tasks에서 확인한 작업이 실행되기까지 소요된 최대 시간
      "active_shards_percent" : "100.0%"	// 정상적으로 동작하는 샤드의 비율
    }
  ]
  ```

  - `relo`
    - 0이 아니라면 샤드들이 재배치 되고 있다는 의미이다.
    - 지나치게 많은 샤드들이 재배치 되고 있을 경우 색인이나 검색 성능에 악영향을 줄 수 있기 때문에 재배치 되는 원인을 확인해 봐야 한다.

  - `init`
    - 0이 아니라면 그만큼의 프라이머리 혹은 레플리카 샤드가 새롭게 배치되고 있다는 의미이다.

  - `pending_tasks`
    - 0이 아니라면 클러스터가 부하 상황이거나 특정 노드가 서비스 불능 상태일 가능성이 있다.
  - `max_task_wait_time`
    - 이 값이 클수록 작업이 실행되지 못하고 오랫동안 큐에 있었다는 뜻이다.
    - 클러스터의 부하 상황을 나타내는 지표로 활용된다.
  - `active_shards_percent`
    - 100%가 아니라면 초기화중이거나 배치되지 않은 샤드가 존재한다는 의미이다.



- 클러스터의 상탯값

  | 값                                       | 의미                                                         |
  | ---------------------------------------- | ------------------------------------------------------------ |
  | <span style="color:green">green</span>   | 모든 샤드가 정상적으로 동작하고 있는 상태                    |
  | <span style="color:orange">yellow</span> | 모든 프라이머리 샤드는 정상적으로 동작하고 있지만, 일부 혹은 모든 레플리카 샤드가 정상적으로 동작하고 있지 않은 상태 |
  | <span style="color:red">red</span>       | 일부 혹은 모든 프라이머리/레플리카 샤드가 정상적으로 동작하고 있지 않은 상태, 데이터 유실이 발생할 수 있다. |



- 미할당 샤드 수동으로 재할당하기

  ```http
  POST _cluster/reroute?retry_failed=true
  ```



### 실습

- 실습을 위한 준비

  - 아래와 같이 3개의 노드로 구성된 cluster를 생성한다.

  ```yaml
  services:
    node1:
      image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
      container_name: node1
      environment:
        - node.name=node1
        - cluster.name=test
        - bootstrap.memory_lock=true
        - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
        - discovery.seed_hosts=node2,node3
        - cluster.initial_master_nodes=node1
        - xpack.security.enabled=false
        - xpack.security.enrollment.enabled=false
      ulimits:
        memlock:
          soft: -1
          hard: -1
      ports:
        - 9200:9200
      
    node2:
      image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
      container_name: node2
      environment:
        - node.name=node2
        - cluster.name=test
        - bootstrap.memory_lock=true
        - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
        - discovery.seed_hosts=node1,node3
        - cluster.initial_master_nodes=node1
        - xpack.security.enabled=false
        - xpack.security.enrollment.enabled=false
      ulimits:
        memlock:
          soft: -1
          hard: -1
    
    node3:
      image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
      container_name: node3
      environment:
        - node.name=node3
        - cluster.name=test
        - bootstrap.memory_lock=true
        - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
        - discovery.seed_hosts=node1,node2
        - cluster.initial_master_nodes=node1
        - xpack.security.enabled=false
        - xpack.security.enrollment.enabled=false
      ulimits:
        memlock:
          soft: -1
          hard: -1
    
    
    kibana:
      image: docker.elastic.co/kibana/kibana:8.11.0
      container_name: kibana
      ports:
        - 5601:5601
      environment:
        ELASTICSEARCH_URL: http://node1:9200
        ELASTICSEARCH_HOSTS: http://node1:9200
  ```

  - Index를 생성한다.

  ```json
  // PUT test
  {
    "settings": {
      "number_of_replicas": 2
    }
  }
  ```

  - Cluster의 상태를 확인한다.

  ```json
  // GET _cluster/health
  
  // response
  {
    "cluster_name": "test",
    "status": "green"
    // ...
  }
  ```

  - Shard의 상태를 확인한다.

  ```json
  // GET _cat/shards/test?h=id,s,p,st,n,ur,ua,uf&format=json
  
  // response
  [
    {
      "id": "pHl7fWw4R2m-kWziWOeKxw",
      "s": "0",
      "p": "r",
      "st": "STARTED",
      "n": "node3",
      "ur": null,
      "ua": null,
      "uf": null
    },
    {
      "id": "--CoQqsLRB-ReNiaNl8C0g",
      "s": "0",
      "p": "p",
      "st": "STARTED",
      "n": "node1",
      "ur": null,
      "ua": null,
      "uf": null
    },
    {
      "id": "Pnf0PTbuTkqWFISc_eGi7w",
      "s": "0",
      "p": "r",
      "st": "STARTED",
      "n": "node2",
      "ur": null,
      "ua": null,
      "uf": null
    }
  ]
  ```



- Primary shard가 할당된 node가 종료될 경우 어떤 일이 발생하는지 관찰하기

  - Primary shard가 할당된 node1을 종료한다.

  ```bash
  $ docker stop node1
  ```

  - Cluster의 상태를 확인한다.
    - status가 "yellow"로 변경된 것을 확인할 수 있다.

  ```json
  // GET _cluster/health
  {
    "cluster_name": "test",
    "status": "yellow",
    // ...
  }
  ```

  - Shard의 상태를 확인한다.
    - 기존에 node1에 할당되었던 shard가 미할당 상태가 되었다.
      - 또한 node3에 있던 replica shard가 primary shard로 승격됐다.

  ```json
  // GET _cat/shards/test?h=s,p,st,n,ur,ua,uf&format=json
  
  
  // response
  [
    {
      "id": "pHl7fWw4R2m-kWziWOeKxw",
      "s": "0",
      "p": "p",
      "st": "STARTED",
      "n": "node3",
      "ur": null,
      "ua": null,
      "uf": null
    },
    {
      "id": null,
      "s": "0",
      "p": "r",
      "st": "UNASSIGNED",
      "n": "null",
      "ur": "NODE_LEFT",
      "ua": "2025-05-14T08:52:22.811Z",
      "uf": "1.4m"
    },
    {
      "id": "Pnf0PTbuTkqWFISc_eGi7w",
      "s": "0",
      "p": "r",
      "st": "STARTED",
      "n": "node2",
      "ur": null,
      "ua": null,
      "uf": null
    }
  ]
  ```

  





## 노드의 상태와 정보 확인하기

- `_cat/nodes`

  - 노드의 상태를 확인하는 API
  - `v`, `format`등의 옵션을 사용 가능하다.
  - `h` 옵션을 사용 가능하다.

  ```bash
  # 아래 명령을 사용하면 h 옵션에서 사용 가능한 값들을 확인 가능하다.
  $ curl -XGET 'http://localhost:9200/_cat/nodes?help'
  
  # h 옵션은 아래와 같이 주면 된다.
  $ curl -XGET 'http://localhost:9200/_cat/nodes?h=id,name,disk.used_percent'
  ```



- 노드 상태 정보

  ```json
  [
    {
      "ip" : "123.123.456.4",	// 노드의 IP 주소
      "heap.percent" : "15",	// 힙 메모리의 사용률
      "ram.percent" : "49",	// 메모리 사용률
      "cpu" : "4",			// 노드의 CPU 사용률
      "load_1m" : "2.11",		// 각각 1,5,15분의 평균 Load Average를 의미한다.
      "load_5m" : "2.21",
      "load_15m" : "2.20",
      "node.role" : "dilm",	// 노드의 역할
      "master" : "-",			// 마스터 여부, 마스터 노드는 *로 표시된다.
      "name" : "node2"		// 노드의 이름
    }
  ]
  ```

  - `heap.percent`
    - 이 값이 크면 클수록 사용 중인 힙 메모리의 양이 많다는 뜻이다.
    - 일정 수준 이상 커지면  old GC에 의해서 힙 메모리의 사용률이 다시 내려간다.
    - 만약 이 값이 낮아지지 않고 85% 이상을 계속 유지하면 OOM(Out Of Memory)이 발생할 가능성이 크기 때문에 힙 메모리가 올라가는 이유를 확인해 봐야 한다.
  - `ram.percent`
    - `heap.percent`가 JVM이 할당받은 힙 메모리 내에서의 사용률이라면 `ram.percent`는 노드가 사용할 수 있는 전체 메모리에서 사용 중인 메모리의 사용률이다.
    - 이 값은 대부분 90% 이상의 높은 값을 나타내는데, JVM에 할당된 힙 메모리 외의 영역은 OS에서 I/O 부하를 줄이기 위한 페이지 캐시로 사용하기 때문이다.

  - `cpu`
    - 이 값이 크면 CPU를 많이 사용하고 있다는 뜻이며, 경우에 따라서는 클러스터에 응답 지연 현상이 발생할 수 있다.

  - `load_nm`
    - 이 값이 크면 노드에 부하가 많이 발생하고 있다는 뜻이며 클러스터의 응답 지연이 발생할 수 있다.
    - Load Average는 노드에 장착된 CPU 코어의 개수에 따라 같은 값이라도 그 의미가 다를 수 있기 때문에 CPU Usage와 함께 살펴보는 것이 좋다.
    - CPU Usage와 Load Average가 모두 높다면 부하를 받고 있는 상황이다.
  - `node.role`
    - d는 데이터, m은 마스터, i는 인제스트, l은 머신러닝을 의미한다.



## 인덱스의 상태와 정보 확인하기

- `_cat/indices`

  - 인덱스의 상태도 green, yellow, red로 표현된다.
    - 인덱스들 중 하나라도 yellow면 클러스터도 yellow, 하나라도 red면 클러스터도 red가 된다.
  - `v`, `h`, `format` 옵션을 모두 사용 가능하다.
  - 확인

  ```http
  GET _cat/indices
  ```

  - `s`로 정렬이 가능하다.

  ```http
  GET _cat/indices?s=<정렬 할 내용>
  
  #e.g.
  GET _cat/indices?s=docs.count:desc
  ```

  - size를 표시할 때 어떤 단위로 보여줄지 설정이 가능하다.
    - `bytes=<단위>`를 입력하면 된다.


  ```http
  GET _cat/indices?h=i,p,r,dc,ss,cds&bytes=kb
  ```

  - `expand_wildcards` 옵션을 통해 open 상태인 인덱스만 보는 것도 가능하다.

  ```http
  GET _cat/indices?h=i,status,p,r,dc,ss,cds&s=cds:desc&expand_wildcards=open
  ```



- 인덱스 상태 정보

  ```json
  // GET _cat/indices?format=json
  [
      {
          "health" : "green",	// 인덱스 상탯값
          "status" : "open",	// 사용 가능 여부
          "index" : "test",	// 인덱스명
          "uuid" : "cs4-Xw3WRM2FpT4bpocqFw",
          "pri" : "1",		// 프라이머리 샤드의 개수
          "rep" : "1",		// 레플리카 샤드의 개수
          "docs.count" : "0",		// 저장된 문서의 개수
          "docs.deleted" : "0",	// 삭제된 문서의 개수
          "store.size" : "566b",		// 인덱스가 차지하고 있는 전체 용량(프라이머리+레플리카)
          "pri.store.size" : "283b"	// 프라이머리 샤드가 차지하고 있는 전체 용량
        }
  ]
  ```





## 샤드의 상태 확인하기

- `_cat/shards`

  - 마찬가지로 `v`, `h`, `format` 모두 사용 가능하다.

  ```http
  GET _cat/shards
  ```

  - `grep`을 사용 가능하다.
    - 아래와 같이 입력하면 state가 `UNASSIGNED`인 샤드만을 반환한다.
    - kibana console에서는 사용 불가

  ```bash
  $ curl -XGET 'http://localhost:9200/_cat/shards | grep UNASSIGNED'
  ```

  - 미할당 샤드가 있을 경우, `h` 옵션을 통해 미할당 된 원인을 확인할 수 있다.
    - `클러스터 운영하기`에서 살펴본  `explain`을 통해서도 확인이 가능하다.
    - 각 원인에 대한 설명은 [공식 가이드](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-shards.html#cat-shards-query-params) 참고

  ```bash
  # 어떤 인덱스의 어떤 샤드가 왜 미할당 상태인지 확인하는 명령어
  GET 'http://localhost:9200/_cat/shards?h=index,shard,prirep,unassigned.reason | grep -i UNASSIGNED'
  ```

  - 보다 정확한 원인을 알려주는 API

  ```http
  GET _cluster/allocation/explain
  ```



- 응답

  ```json
  [
  	{
          "index" : "test",		// 샤드가 속한 인덱스명
          "shard" : "0",			// 샤드 번호
          "prirep" : "p",			// 프라이머리인지 레플리카인지(레플리카는 r)
          "state" : "STARTED",	// 샤드의 상태
          "docs" : "0",			// 샤드에 저장된 문서의 수
          "store" : "283b",		// 샤드의 크기
          "ip" : "123.456.789.1",	// 샤드가 배치된 노드의 IP
          "node" : "node2"		// 샤드가 배치된 데이터 노드의 노드 이름
    	}
  ]
  ```

  - state

  | 값           | 의미                                                         |
  | ------------ | ------------------------------------------------------------ |
  | STARTED      | 정상적인 상태                                                |
  | INITIALIZING | 샤드를 초기화하는 상태. 최초 배치 시, 혹은 샤드에 문제가 발생하여 새롭게 배치할 때의 상태 |
  | RELOCATING   | 샤드가 현재의 노드에서 다른 노드로 이동하고 있는 상태. 새로운 데이터 노드가 추가되거나, 기존 데이터 노드에 문제가 생겨서 샤드가 새로운 노드에 배치되어야 할 때의 상태 |
  | UNASSIGNED   | 샤드가 어느 노드에도 배치되지 않은 상태. 해당 샤드가 배치된 노드에 문제가 생기거나 클러스터의 라우팅 정책에 의해 배치되지 않은 상태. |



- segment 상태 확인

  - `_cat` API
    - `v`, `h`, `format` 옵션을 모두 사용 가능하다.


  ```http
  GET _cat/segments[/<target>]
  ```

  - Index segments API
    - 특정 index에 속한 segments들만 확인할 수 있다.
    - Application에서 사용할 때는 이 API를 사용하는 것이 권장된다.

  ```http
  GET <index_name>/_segments
  ```

  - Index segments API의 response
    - Segment의 이름은 실제 file이름이다(`<data_dir>/indices/<index_uuid>/<shard>/index`에서 확인할 수 있다).
    - `num_docs`은 삭제된 문서는 포함하지 않으며, nested documents들도 별개의 문서로 집계한다.
    - `committed`가 true면 disk와 segment가 sync되었다는 것을 의미하며, false면 disk와 sync되어 있지 않다는 의미이다.
    - Disk와 sync되어 있다면(committed가 true면) ES가 reboot 되더라도 segment data에 유실이 없다.
    - `committed`가 false라고 하더라도, commit되지 않은 segment의 data들은 trans log에 기록되어 ES가 reboot될 때 trans log에서 data를 읽어올 수 있으므르로 유실이 발생할 일은 거의 없다.
    - `search`가 true면 검색이 가능하다는 의미이고, false이면 refresh를 통해 검색이 가능하게 해줘야한다는 의미이다.
    - `version`은 Lucene version을 의미한다.
    - `attributes`: 압축과 관련된 정보를 보여준다.

  ```json
  "shards": {
      "0": [
          {
              "routing": {
                  "state": "STARTED",
                  "primary": false,
                  "node": "5g4483g4g34gg5123T123"
              },
              "num_committed_segments": 1,
              "num_search_segments": 1,
              "segments": {
                  // segment의 이름
                  "_1v": {
                      "generation": 67,			// 새로운 segment가 생성될 때 마다 증가하는 숫자, segment name을 결정할 때 사용한다.
                      "num_docs": 1000000,		// segment에 저장된 문서의 수
                      "deleted_docs": 0,			// 삭제된 문서의 수(실제 삭제된 문서의 개수와 다를 수 있다)
                      "size_in_bytes": 1598215025,// segment의 크기
                      "committed": true,
                      "search": true,
                      "version": "9.4.2",
                      "compound": false,
                      "attributes": {
                          "Lucene90StoredFieldsFormat.mode": "BEST_SPEED"
                      }
                  }
              }
          }
      ]
  }
  ```

  - `compound`
    -  true면 Lucene이 file descriptor를 아끼기 위해 segment의 모든 파일들을 하나의 파일에 저장했다는 의미이다.
    -  Lucene에서 segment는 compound 방식과 multifile 방식이라는 두 가지 방식 중 하나로 저장된다.
    -  Multifile 방식은 segment를 여러 개의 file을 사용해서 저장하는 방식으로, term vector, inverted index, stored field 등을 모두 개별 file에 작성한다.
    -  Multifile의 단점은 너무 많은 file을 관리해야 한다는 점이다. OS에서는 open file의 개수가 제한되어 있는데(linux의 경우 `ulimit -n <숫자>` 옵션으로 변경 가능), 너무 많은 segment file이 열려 있을 경우 `Too many open files` error가 발생할 수 있다.
    -  Compound 방식은 segment의 여러 file들(term vector, inverted index, stored field 등)을 하나의 file에 저장하는 방식으로 file descriptor를 multifile 방식에 비해 덜 필요로 한다는 장점이 있다.




## stats API로 지표 확인하기

- GC(Garbage Collector)

  - 정의

    - Java로 만든 애플리케이션은 기본적으로 JVM이라는 가상 머신 위에서 동작하는데 OS는 JVM이 사용할 수 있도록 일정 크기의 메모리를 할당해준다.
    - 이 메모리 영역을 힙 메모리라고 부른다.
    - JVM은 힙 메모리 영역을 데이터를 저장하는 용도로 사용한다.
    - 시간이 갈수록 사용 중인 영역이 점점 증가하다가 어느 순간 사용할 수 있는 공간이 부족해지면 사용 중인 영역에서 더 이상 사용하지 않는 데이터들을 지워서 공간을 확보하는데 이런 일련의 과정을 가비지 컬렉션이라 부른다.

    - 사용 중인 영역은 young 영역과  old 영역 두 가지로 나뉜다.

  - Stop-The-World

    - GC 작업을 할 때, 즉 메모리에서 데이터를 지우는 동안에는 다른 스레드들이 메모리에 데이터를 쓰지 못하도록 막는다.
    - GC가 진행되는 동안에너는 다른 스레드들이 동작하지 못하기 때문에 애플리케이션이 응답 불가 현상을 일으키고 이를 Stop-The-World 현상이라 한다.
    - 특히 old GC는 비워야 할 메모리의 양이 매우 많기 때문에 경우에 따라서는 초 단위의 GC 수행 시간이 소요되기도 한다.
    - 이럴 경우 ES 클러스터가 초 단위의 응답 불가 현상을 겪게 된다.

  - Out Of Memory

    - GC를 통해 더 이상 메모리를 확보할 수 없는 상황에서 애플리케이션이 계속해서 메모리를 사용하고자 하면 가용할 메모리가 없다는 OOM 에러를 발생시킨다.
    - OOM  에러는 애플리케이션을 비정상 종료시키기 때문에 클러스터에서 노드가 아예 제외되는 현상이 발생한다.



- 클러스터 성능 지표 확인하기

  - 명령어

  ```bash
  $ curl -XGET 'http://localhost:9200/_cluster/stats?pretty
  ```

  - 성능 지표

  ```json
  {
    "_nodes" : {
      "total" : 4,
      "successful" : 4,
      "failed" : 0
    },
    "cluster_name" : "es-docker-cluster",
    "cluster_uuid" : "bOfgi147StyhjT2sOzdPYw",
    "timestamp" : 1620780055464,
    "status" : "green",
    "indices" : {
      // (...)
      "docs" : {
        "count" : 9,		// 색인된 전체 문서의 수
        "deleted" : 1		// 삭제된 전체 문서의 수
      },
      "store" : {
        "size_in_bytes" : 140628	// 저장 중인 데이터의 전체 크기를 bytes 단위로 표시
      },
      "fielddata" : {
        "memory_size_in_bytes" : 0,	// 필드 데이터 캐시의 크기
        "evictions" : 0
      },
      "query_cache" : {
        "memory_size_in_bytes" : 0,	// 쿼리 캐시의 크기
        // (...)
      },
      "completion" : {
        "size_in_bytes" : 0
      },
      "segments" : {
        "count" : 17,		// 세그먼트의 수
        "memory_in_bytes" : 28061,	// 세그먼트가 차지하고 있는 메모리의 크기
        // (...)
      }
    },
    // (...)
      "versions" : [	// 클러스터를 구성하고 있는 노드들의 버전
        "7.5.2"
      ],
      // (...)
      "jvm" : {
        "max_uptime_in_millis" : 69843238,
        "versions" : [	// 클러스터를 구성하고 있는 노드들의 JVM 버전
          {
            "version" : "13.0.1",
            "vm_name" : "OpenJDK 64-Bit Server VM",
            "vm_version" : "13.0.1+9",
            "vm_vendor" : "AdoptOpenJDK",
            "bundled_jdk" : true,
            "using_bundled_jdk" : true,
            "count" : 4
          }
        ],
     // (...)
  }
  ```

  - `fielddata.memory_size_in_bytes`
    - 필드 데이터는 문자열 필드에 대한 통계 작업을 할 때 필요한 데이터이다.
    - 필드 데이터의 양이 많으면 각 노드의 힙 메모리 공간을 많이 차지하기 때문에 이 값이 어느 정도인지 모니터링해야 한다.
    - 노드들의 힙 메모리 사용률이 높다면 우선적으로 필드 데이터의 유무를 확인하는 것이 좋다.

  - `query_cache.memory_size_in_bytes`
    - 모든 노드들은 쿼리의 결과를 캐싱하고 있다.
    - 이 값이 커지면 힙 메모리를 많이 차지하기에 이 값이 어느정도인지 모니터링해야 한다.

  - `segments.memory_in_bytes`
    - 세그먼트도 힙 메모리 공간을 차지하기 때문에 힙 메모리의 사용률이 높을 경우 세그먼트의 메모리가 어느 정도를 차지하고 있는지 살펴봐야 한다.
    - forcemerge API를 사용하여 세그먼트를 강제 병합하면 세그먼트의 메모리 사용량도 줄일 수 있다.



- 노드의 성능 지표

  - 명령어

  ```bash
  $ curl -XGET 'http://localhost:9200/_nodes/stats?pretty
  ```

  - 성능 지표

  ```json
  {
    "_nodes" : {
      "total" : 4,
      "successful" : 4,
      "failed" : 0
    },
    "cluster_name" : "es-docker-cluster",
    "nodes" : {
      "rrqnQp2mRUmPh8OGqTj0_w" : {	// 노드의 ID, 클러스터 내부에서 임의의 값을 부여한다.
        "timestamp" : 1620781452291,
        "name" : "node2",				// 노드의 이름
        "transport_address" : "192.168.240.5:9300",
        "host" : "192.168.240.5",
        "ip" : "192.168.240.5:9300",
        "roles" : [		// 노드가 수행할 수 있는 역할
          "ingest",
          "master",
          "data",
          "ml"
        ],
        "attributes" : {
          "ml.machine_memory" : "134840213504",
          "ml.max_open_jobs" : "20",
          "xpack.installed" : "true"
        },
        "indices" : {
          "docs" : {
            "count" : 9,		// 노드가 지니고 있는 문서의 수
            "deleted" : 1
          },
          "store" : {
            "size_in_bytes" : 70928	// 노드가 저장하고 있는 문서의 크기를 byte 단위로 표시
          },
          "indexing" : {
            "index_total" : 23,		// 지금까지 색인한 문서의 수
            "index_time_in_millis" : 192,	// 색인에 소요된 시간
            // (...)
          },
          "get" : {			// REST API의 GET 요청으로 문서를 가져오는 성능에 대한 지표(검색 성능보다는 문서를 가져오는 성능을 의미한다)
            // (...)
          },
          "search" : {		// 검색 성능과 관련된 지표
            // (...)
          },
          "merges" : {		// 세그먼트 병합과 관련된 성능 지표
            // (...)
          },
          // (...)
          "query_cache" : {		// 쿼리 캐시와 관련된 지표
            // (...)
          },
          "fielddata" : {			// 필드 데이터 캐시와 관련된 지표
            "memory_size_in_bytes" : 0,
            "evictions" : 0
          },
          "completion" : {
            "size_in_bytes" : 0
          },
          "segments" : {			// 노드에서 사용중인 세그먼트와 관련된 지표
            // (...)
          },
          // (...)
        },
        "os" : {
          "timestamp" : 1620781452298,
          "cpu" : {
            "percent" : 3,	// 노드의 CPU 사용률
            "load_average" : {	// 노드의 Load Average 사용률(각 1,5,15분 평균을 의미)
              "1m" : 1.99,
              "5m" : 1.9,
              "15m" : 1.9
            }
          },
          // (...)
          "gc" : {		// GC와 관련된 성능 지표
            "collectors" : {
              "young" : {
                "collection_count" : 317,
                "collection_time_in_millis" : 1595
              },
              "old" : {
                "collection_count" : 3,
                "collection_time_in_millis" : 53
              }
            }
          },
          // (...)
        },
        "thread_pool" : {		// 노드의 스레드 풀 상태
          // (...)
          "search" : {
            "threads" : 73,
            "queue" : 0,
            "active" : 0,
            "rejected" : 0,
            "largest" : 73,
            "completed" : 10917
          },
          // (...)
          "write" : {
            "threads" : 32,
            "queue" : 0,
            "active" : 0,
            "rejected" : 0,
            "largest" : 32,
            "completed" : 32
          }
        },
        "fs" : {
          "timestamp" : 1620781452302,
          "total" : {		// 디스크의 사용량을 의미한다.
            "total_in_bytes" : 1753355112448,
            "free_in_bytes" : 1117627813888,
            "available_in_bytes" : 1028490833920	// 현재 남아 있는 용량
          },
          // (...)
      }
    }
  }
  ```

  - `indexing.index_total`
    - 카운터 형식의 값으로 0부터 계속해서 값이 증가한다.
    - 지금 호출한 값과 1분 후에 호출한 값이 차이가 나며 이 값의 차이가 1분 동안 색인된 문서의 개수를 나타낸다.
    - 색인 성능을 나타내는 매우 중요한 지표 중 하나이다.

  - `thread_pool`
    - 검색에 사용하는 search 스레드, 색인에 사용하는 write 스레드 등  스레드들의 개수와 큐의 크기를 보여준다.
    - rejected가 매우 중요한데, 현재 노드가 처리할 수 있는 양보다 많은 요청이 들어오고 있기 때문에 더 이상 처리할 수 없어서 처리를 거절한다는 의미이기 때문이다.

 

- 인덱스 성능 지표

  > https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-stats.html

  - 위 링크를 확인해보면 각종 parameter들이 있는데 확인을 원하는 파라미터를 입력하면 된다.
  - 쿼리 수, 쿼리 평균 시간 등도 확인 가능하다.

  ```bash
  $ curl -XGET 'http://localhost:9200/<인덱스명>/_stats'
  ```



## tasks API로 task 확인하기

- 진행중인 task 확인

  ```bash
  # 특정 task 조회
  $ curl -XGET 'http://localhost:9200/_tasks/<task_id>'
  
  # 전체 task 조회
  $ curl -XGET 'http://localhost:9200/_tasks'
  ```



- 파라미터

  - actions
    - 콤마로 구분 된 task명이나, 와일드카드 표현식을 사용해서 task를 필터링 할 때 사용한다.

  ```bash
  # 예시
  $ curl -XGET 'http://localhost:9200/_tasks?actions=*bulk'
  ```

  - detalied
    - Boolean 값을 주며, true를 줄 경우 shard recoveries에 대한상세 정보를 함께 반환한다.
    - 기본값은 false이다.

  ```bash
  $ curl -XGET 'http://localhost:9200/_tasks?actions=*bulk&detailed=true'
  ```

  - group_by
    - task들을 grouping하기 위해 사용한다.
    - nodes(기본값): Node ID로 그룹핑한다.
    - parents: parent task ID로 그룹핑한다.
    - none: 그룹핑하지 않는다.

  ```bash
  $ curl -XGET 'http://localhost:9200/_tasks?group_by=parents'
  ```

  - node_id
    - 콤마로 구분 된 노드 ID 또는 노드 이름을 인자로 받는다.
    - 리스트에 포함된 노드의 task만 반환한다.
  - parent_task_id
    - 입력한 Parent task ID에 해당하는 task들만 반환한다.
    - 이 파라미터를 넘기지 않거나 -1을 값으로 주면 모든 task를 반환한다.
  - master_timeout
    - 마스터 노드에 연결되기까지의 기간(기본값은 30s)을 설정한다.
    - 기간 내에 응답이 오지 않으면 errror를 반환한다.
  - wait_for_completion
    - Boolean값을 받으며, true로 줄 경우 operation이 완료될 때 까지 요청이 중단된다.
    - 기본값은 false이다.





## 성능 확인과 문제 해결

- 주요 성능
  - 색인 성능
    - 초당 몇 개의 문서를 색인할 수 있는지, 그리고 각 문서를 색인하는 데 소요되는 시간이 어느 정도인지
    - 클러스터 전체 성능과 노드의 개별 성능으로 나누어 측정한다.
  - 검색 성능
    - 초당 몇 개의 쿼리를 처리할 수 있는지, 그리고 각 쿼리를 처리하는 데 소요되는 시간이 어느 정도인지
    - 클러스터 전체 성능과 노드의 개별 성능으로 나누어 측정한다.
  - GC 성능
    - GC가 너무 자주, 오래 발생하면 Stop-The-World 같은 응답 불가 현상이 발생한다.
    - Stop-The-World가 얼마나 자주, 오래 발생하는지를 나타낸다.
  - rejected
    - 클러스터가 처리할 수 없는 수준의 요청이 들어오면 클러스터는 요청을 거절하게 되는데 그 횟수를 나타낸다.



- 색인 성능

  - `_stats` 를 통해 확인 가능하다.

  ```bash
  $ curl -XGET 'http://localhost:9200/_stats?pretty
  ```

  - 성능 지표

  ```json
  {
    "_shards" : {
      "total" : 14,
      "successful" : 14,
      "failed" : 0
    },
    "_all" : {
      "primaries" : {		// 프라이머리 샤드에 대한 색인 성능
        "docs" : {
          "count" : 10,
          "deleted" : 1
        },
        "store" : {
          "size_in_bytes" : 92866
        },
        "indexing" : {
          "index_total" : 24,
          "index_time_in_millis" : 209,
          // (...)
        },
        // (...)
      "total" : {		// 전체 샤드에 대한 색인 성능
       // 내용은 primaries와 동일하다.
       //(...)
      }
    }
  }
  ```

  - 성능 측정
    - 위 요청을 10초 간격으로 보냈을 때, 첫 번째 호출시 색인된 문서의 수는 0개, 두 번째 호출시 색인된 문서의 수는 200개 라고 가정
    - 10 초 동안 200개의 문서가 색인됐다고 할 수 있다.
    - 또한 첫 번째 호출시 색인 소요 시간이 0초, 두 번째 호출 시 색인 소요 시간이 100ms라고 가정
    - 즉 10초 동안 200개의 문서를 색인했고, 색인하는데 10ms가 걸렸기 때문에 각각의 문서를 색인하는데 에는 200/100=2ms의 시간이 소요되었음을 알 수 있다.
    - 즉, 이 클러스터의 프라이머리 샤드에 대한 색인 성능은 2ms이다.
    - 색인하는 양이 많으면 많을수록 색인에 소요되는 시간도 늘어나기 때문에 두 값의 절대적인 값보다는 하나의 문서를 색인하는 데 시간이 얼마나 소요되는지를 더 중요한 성능 지표로 삼아야 한다.



- 검색 성능

  - query와 fetch
    - A, B, C 노드가 있을 때 사용자가 search API를 통해 A 노드에 검색 요청을 입력했다고 가정한다.
    - 그럼 노드 A는 자신이 받은 검색 쿼리를 B, C 노드에 전달한다.
    - 각각의 노드는 자신이 가지고 있는 샤드 내에서 검색 쿼리에 해당하는 문서가 있는지 찾는 과정을 진행한다.
    - 이 과정이 query이다.
    - 그리고 이렇게 찾은 문서들을 리스트 형태로 만들어서 정리하는 과정이 fetch이다.
    - 검색은 이렇게 query와 fetch의 과정이 모두 끝나야 만들어지기 때문에 검색 성능을 측정할 때 두 과정을 모두 포함하는 것이 좋다.
  - `_stats`을 통해 확인 가능하다.

  ```bash
  $ curl -XGET 'http://localhost:9200/_stats?pretty
  ```

  - 성능 지표

  ```json
  {
    "_shards" : {
      "total" : 14,
      "successful" : 14,
      "failed" : 0
    },
    "_all" : {
      // (...)
        "search" : {
          "open_contexts" : 0,
          "query_total" : 10916,	// 호출하는 시점까지 처리된 모든 query의 총합
          "query_time_in_millis" : 5496,
          "query_current" : 0,
          "fetch_total" : 10915,	// 호출하는 시점까지 처리된 모든 fetch의 총합
          "fetch_time_in_millis" : 476,
          "fetch_current" : 0,
          "scroll_total" : 9741,
          "scroll_time_in_millis" : 18944,
          "scroll_current" : 0,
          "suggest_total" : 0,
          "suggest_time_in_millis" : 0,
          "suggest_current" : 0
        },
      // (...)
      }
    }
  }
  ```

  - 성능 측정
    - 색인 성능 측정과 동일한 방식으로 진행하면 된다.
    - query와 fetch를 나눠서 진행한다.



- GC 성능 측정

  - 각 노드에서 발생하기 때문에 `_nodes/stats`를 통해 확인한다.

  ```bash
  $ curl -XGET 'http://localhost:9200/_nodes/stats?pretty
  ```

  - 성능 지표

  ```json
  // (...)
  "jvm" : {
      // (...)
    "gc" : {
      "collectors" : {
          "young" : {
              "collection_count" : 333,
              "collection_time_in_millis" : 1697
          },
          "old" : {
              "collection_count" : 3,
              "collection_time_in_millis" : 53
          }
      }
  }
  // (...)
  ```

  - 성능 측정
    - 색인과 동일한 방법으로 측정한다.
    - old, young을 각각 측정한다.
    - 상황에 따라 다르지만 보통 수십에서 수백 ms 정도의 성능을 내는 것이 안정적이다.



- rejected 성능 측정

  - rejected 에러
    - ES는 현재 처리할 수 있는 양보다 많은 양의 요청이 들어올 경우 큐에 요청을 쌓아놓는다.
    - 하지만 큐도 꽉 차서 더 이상 요청을 쌓아놓을 수 없으면 rejected 에러를 발생시키며 요청을 처리하지 않는다.
    - 보통 요청이 점차 늘어나서 초기에 구성한 클러스터의 처리량이 부족한 경우나, 평상시에는 부족하지 않지만 요청이 순간적으로 늘어나서 순간 요청을 처리하지 못하는 경우에 발생한다.
  - node별로 측정이 가능하다.

  ```bash
  $ curl -XGET 'http://localhost:9200/_nodes/stats?pretty
  ```

  - 성능 지표
    - 각 스레드 별로 확인할 수 있다.

  ```json
  // (...)
  "thread_pool" : {		// 노드의 스레드 풀 상태
      // (...)
      "search" : {
        "threads" : 73,
        "queue" : 0,
        "active" : 0,
        "rejected" : 0,
        "largest" : 73,
        "completed" : 10917
      },
      // (...)
      "write" : {
        "threads" : 32,
        "queue" : 0,
        "active" : 0,
        "rejected" : 0,
        "largest" : 32,
        "completed" : 32
      }
  },
  // (...)
  ```

  - rejected 문제 해결
    - 만일 초기에 구성한 클러스터의 처리량이 부족하다면 데이터 노드를 증설하는 방법 외에는 특별한 방법이 없다.
    - 그러나 순간적으로 밀려들어오는 요청을 처리하지 못한다면 큐를 늘리는 것이 도움이 될 수 있다.
    - ` elasticsearch.yml` 파일에 아래와 같이 설정하면 된다.

  | 스레드 이름         | 스레드 풀 타입        | 설정 방법                                | 예시                                    |
  | ------------------- | --------------------- | ---------------------------------------- | --------------------------------------- |
  | get, write, analyze | fixed                 | thread_pool.[스레드 이름].queue_size     | thread_pool.write.queue_size=10000      |
  | search              | fixed_auto_queue_size | thread_pool.[스레드 이름].max_queue_size | thread_pool.search.max_queue_size=10000 |







# Shorts

- 사용자가 입력한 검색어를 Elasticsearch analyzer가 아닌 곳에서 변형하는 것이 옳은가?

  - 일반적으로 사용자가 입력한 검색어가 Elasticsearch로 갈 때까지는 몇 개의 단계를 거치게 된다.
    - 예를 들어, 사용자가 검색 화면에서 검색어를 입력할 경우 web application이 해당 검색어를 받아 server API로 전송한다.
    - Server API는 client의 요청에 따라 Elasticsearch query를 구성하여, Elasticsearch에 검색 요청을 전송한다.
  - 예를 들어 아래와 같은 index가 있다고 가정해보자.

  ```json
  // PUT my-index
  {
      "settings": {
          "analysis": {
              "tokenizer": {
                  "my_nori_tokenizer":{
                      "type":"nori_tokenizer",
                      "user_dictionary_rules":[
                          "K리그"
                      ]
                  }
              },
              "analyzer": {
                  "my_analyzer":{
                      "type": "custom",
                      "tokenizer": "my_nori_tokenizer"
                  }
              }
          }
      },
      "mappings": {
          "properties": {
              "content":{
                  "type":"text",
                  "analyzer": "my_analyzer"
              }
          }
      }
  }
  ```

  - 아래와 같은 문서를 색인한다.

  ```json
  // PUT my-index/_doc/1
  {
    "content":"김철수는 K리그 최고의 감독이다."
  }
  ```

  - 사용자가 화면에서 검색어를 입력하고 검색 버튼을 누르면 web application은 API server로 검색 요청를 보낸다.
    - API server의 host를 `my-search-api`라고 가정한다.
    - 문제는 이 때, web application이 사용자가 입력한 검색어에 lowercase를 적용한다는 것이다.

  ```json
  // POST my-search-api/v1/search
  {
      "query":"k리그"
  }
  ```

  - Web application으로부터 검색 요청을 받은 API server는 요청을 Elasticsearch query로 변환해서 Elasticsearch로 검색 요청을 보낸다.

  ```json
  // my-index/_search
  {
      "query":{
          "match":{
              "content":"k리그"
          }
      }
  }
  ```

  - 이 처럼 최종적으로 검색을 수행하는 Elasticsearch가 아닌 web application이나 API server에서 검색어를 수정하면 예상치 못 한 문제가 발생할 수 있다.
    - 위 예시에서 소문자로 변환 된 "k리그"라는 검색어를 입력 받은 Elasticsearch는 ["k", "리그"]로 token을 생성한다.
    - 반면에 검색 대상 문서에 포함된 "K리그"는 사용자 사전에 등록되어 있으므로, ["K리그"]와 같이 token이 생성된다.
    - 따라서 사용자는 분명히 "K리그"로 검색했음에도, "K리그"가 포함 된 문서가 검색되지 않는 문제가 발생한다.

  - Elasticsearch 이외의 곳에서 임의로 검색어를 수정했을 때 발생할 수 있는 가장 큰 문제는, 문제의 원인을 추적하기가 힘들다는 점이다.
    - 만약 위 예시에서 web application과 API server의 개발, 그리고 Elasticsearch index 설정을 모두 한 사람이 했다고 가정해보자.
    - 이 사람은 다행이 자신이 web application에서 API server로 요청을 보낼 때, lowercase를 적용했다는 것을 기억해냈고, 이 부분을 수정하여 문제를 해결했다.
    - 그러나 만약 세 부분을 모두 다른 사람이 개발했다고 가정해보자.
    - 사용자는 "K리그"로 검색을 했고, 분명 "K리그"가 포함된 문서가 있음에도 검색되지 않는다고 문제를 보고한다.
    - 처음에는 Elasticsearch의 analyzer에 문제가 있을 것이라 생각하여, 문제가 보고 된 "K리그"라는 검색어로 Elasticsearch에서 직접 검색을 실행한다.
    - "K리그"로 검색을 하면 정상적으로 검색 결과가 나올 것이므로 Elasticsearch에는 이상이 없는 것으로 판단하고, API server에서 Elasticsearch로 넘기는 query에 이상이 있는지 점검한다.
    - 점검 결과 API server에도 이상이 없는 것으로 밝혀져, 마지막으로 web application을 살펴보고, API server로 요청을 전송할 때 검색어에 lowercase를 적용하는 것을 발견하게 된다.
    - 예시에서는 오직 web application에서만 검색어를 수정한다고 가정했지만, 여러 단계를 거치며 각 단계에서 모두 검색어의 수정을 수행한다면, 검색 결과에 문제가 있을 때 원인을 추적하기가 점점 더 힘들어질 것이다.