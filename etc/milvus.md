# Milvus

## 개요

- Milvus
  - 오픈 소스로 개발된 벡터 데이터베이스이다.
    - Zilliz에 의해 개발되었다.
    - Apache 2.0 라이선스로 배포된다.
  - Milvus의 특징
    - IVF, HNSW, DiskANN 등 다양한 검색 알고리즘을 지원한다.
    - FAISS나 HNSWLib과 같은 널리 사용되는 구현체 대비 30~70% 더 우수한 성능을 보인다.
    - 핵심 컴포넌트를 C++로 구현하여 성능이 뛰어나다.



- Architecture

  - 개요
    - 데이터 플레인(data plane)과 컨트롤 플레인(control plane)을 분리(disaggregation)하는 원칙을 따른다.
    - 확장성과 장애 복구 측면에서 서로 독립적으로 동작하는 네 개의 주요 계층으로 구성된다.

  ![Architecture_diagram](milvus.assets/milvus_architecture_2_6.png)

  - Layer1: Access layer

    - Proxy들의 그룹으로 구성된 layer이다.
    - 이 layers는 가장 앞단(front)에 위차한 layer이며 사용자들의 endpoint이다.
    - 사용자의 요청이 유효한지 검사하고, 응답을 후처리하는 역할을 한다.

  - Layer2: Coordinator layer

    - Milvus의 두뇌에 해당하는 layer이다.
    - Cluster 내에 단 하나의 coordinator만이 동작한다.
    - Cluster의 topology를 유지하고, 모든 종류의 작업을 스케줄링하며, 클러스터 단위의 일관성을 보장하는 역할을 담당한다.
    - DDL(data definition language)/DCL(data control language)/TSO(timestamp oracle)을 관리한다.
    - Write-Ahead Log(WAL)을 Streaming Node와 연결하고, 스트리밍 서비스에 대한 서비스 디스커버리를 제공한다.
    - Query Node의 토폴로지와 로드 밸런싱을 관리하며, 쿼리 라우팅을 안내하기 위한 서비스용 쿼리 뷰를 제공하고 관리한다.
    - 압축(compaction), 인덱스 생성과 같은 오프라인 작업을 Data Node에 분배하고, 세그먼트 및 데이터 뷰의 topology를 관리한다.

  - Layer3: Worker nodes layer

    - Coordinator의 지시에 따라 작업을 수행하는 layer이다.
    - Milvus는 별도의 storage를 사용하기 때문에 worker node들은 stateless하다.
    - 아래와 같은 세 종류의 worker node가 있다.
    - Streaming Node: WAL storage를 기반으로 샤드 단위의 일관성 보장과 장애 복구를 담당한다. 또한 growing data querying과 query plans 생성도 담당한다.

    - Query Node: Historical data를 object storage로부터 가져오는 역할을 한다.
    - Data Node: Compaction과 index building 같은 historical data의 offline 처리를 담당한다.

  - Layer4: Storage layer

    - Data를 영구히 저장하는 layer로, meta storage, log broker, object storage가 포함된다.
    - Meta storage는 collection schema나 message consumption checkpoints 같은 metadata의 snapshot을 저장하며, etcd가 meta storage로 사용된다.
    - Object storage는 query 결과, scalar와 vector data의 index file, log file들의 snapshot을 저장하며, 기본적으로 S3를 사용하지만 Azure Blob이나 AWS S3도 사용이 가능하다.
    - WAL storage는 data의 변경 사항이 commit 되기 전에 변경 사항을 기록한 내역을 저장하며, Kafka, Pulsar,  Woodpecker를 사용할 수 있다.

  - Data 삽입 flow

    - Clinet가 vector data 삽입 요청을 전송한다.
    - Access Layer가 요청의 유효성을 검증하고 streaming node로 요청을 전달한다.
    - Streaming Node는 삽입 operation에 대한 log를 WAL Storage에 남긴다.
    - Data는 real-time으로 처리되고, 검색 가능한 상태가 된다.
    - Segment 수용량이 한계에 달하면 streaming node는 해당 segment의 상태를 sealed segment로 전환한다.
    - Data node는 sealed segment들에 대해 compaction과 index 생성을 실행하고, 그 결과를 object storage에 저장한다.
    - Query Node는 새로 생성된 인덱스를 로드하고, 기존의 growing data를 해당 인덱스로 교체한다.

  - 검색 flow

    - Clinet가 SDK/RESTful API 등을 통해 검색 요청을 보낸다.
    - Load Balancer는 현재 요청을 처리할 수 있는 Proxy(access layer)로 요청을 라우팅한다.
    - Proxy는 라우팅 캐시를 사용하여 요청을 전달할 node를 결정한다(만약 cache를 사용할 수 없다면 coordinator와 통신한다).
    - Proxy는 적절한 streaming node로 요청을 전달한다.
    - Streaming node는 로컬에서 growing data 검색을 수행하는 동시에 sealed data 검색을 위해 query node들과 협력한다.
    - Query node는 필요한 경우 object storage에서 sealed segment들을 load하여 segment level 검색을 수행한다.
    - 검색 결과는 여러 단계의 축약 과정을 거친 후에 반환되는데, query node가 여러 세그먼트 간의 결과를 1차 축약하고, streaming node가 query node로부터 받은 결과를 2차 축약하며, proxy가 모든 streaming node의 결과를 최종 축약한 뒤 클라이언트에게 반환한다.



- 데이터 삽입

  - 구조
    - Milvus에서 shard는 virtual channel(vchannel)과 매핑된다.
    - 각각의 vchannel에는 physical channel(pchannel)이 할당되며, 하나의 pchannel이 여러 개의 vchannel과 연결될 수 있다.
    - 각각의 pchannel은 하나의 streaming node에 bound된다.

  ![VChannel PChannel and StreamingNode](milvus.assets/pvchannel_wal.png)

  - 과정
    - 요청에 대한 유효성 검증이 완료되면 proxy는 요청으로 들어온 데이터를 여러 개의 data package로 나눠 shard로 전달한다.
    - 데이터를 전달 받은 shard(vchannel)는 자신과 연결되어 있는 pchannel과 bound된 streaming node로 데이터를 전송한다.
    - Streaming node는 각 data packet에 순서를 매기기 위해 TSO(timestamp oracle)을 할당한다.
    - Streaming node는 WAL에 데이터를 작성하기 전에 일관성 검사를 실행한 후, 문제가 없으면 WAL에 commit한다.
  - Segment의 타입
    - Growing segment: Object storage에 영구적으로 저장되지 않은 데이터들이다.
    - Sealed segment: Object storage에 영구적으로 저장된 데이터들로, 수정이 불가능하다.
  - Flush
    - Growing segment에서 sealed segment로 전환되는 과정을 의미한다.
    - Streaming node는 더 이상 기록해야 할 WAL 항목이 남아있지 않으면 flush를 trigger한다.
    - 이 시점에서 세그먼트는 최종 확정되며 읽기 최적화 상태가 된다.



- Index Building
  - Index building은 data node가 수행하며, data가 update될 때마다 index building이 발생하는 것을 피하기 위해 collection은 segment 단위로 분할된다.
  - Milvus는 각 vector field, scalar field, primary field에 대한 index building을 지원한다. 
    - Index building의 입력과 출력은 모두 object storage를 사용한다. 
    - Data node는 object storage에 저장된 segment부터 인덱싱할 log snapshot을 memory로 load하고, 해당 데이터와 메타데이터를 역직렬화해 index를 생성한다. 
    - Index building이 완료되면 이를 직렬화해 다시 object storage에 저장한다.
  - Index Building은 vector 및 행렬 연산을 주로 포함하므로, 계산량이 많고 메모리 사용량이 큰 작업이다. 
    - 고차원 vector 는 전통적인 tree 기반 index로는 효율적으로 관리할 수 없기 때문에, cluster 기반 또는 그래프 기반 index 등의 기법이 사용된다. 
    - Index 유형과 관계없이, 대규모 vector에 대한 index building은 K-means나 그래프 탐색과 같은 대규모 반복 계산을 포함한다.
    - 스칼라 데이터 인덱싱과 달리, 벡터 인덱스 생성은 SIMD(single instruction, multiple data) 가속을 최대한 활용해야 한다.
    - Milvus는 SSE, AVX2, AVX512 등의 SIMD 명령어 세트를 기본적으로 지원한다. 



- Data query
  - Data query는 target vector와 가장 가까운 k개의 vector를 찾거나 target vector와 일정 거리 내에 있는 모든 vector를 찾는 과정을 말한다.
    - Vector들은 자신들의 primary key와 field들과 함께 반환된다.
    - Milvus에서 collection은 여러 개의 segment로 분할된다.
    - Streaming node는 growing segment들을 load하고 real-time data를 유지하며, query node는 sealed segment들을 load한다.
  - Query가 실행되는 과정
    - Proxy에 검색 요청이 도착하면, proxy는 현재 요청과 관련 있는 shard와 연결된 모든 streaming node에 요청을 전송하여 동시에 검색하도록한다.
    - 각 streaming node는 query plan을 생성하고, 이를 토대로 local growing data에 검색을 수행함과 동시에 query node와 통신해 historical results를 받아온 뒤 이를 하나의 shard result로 통합한다.
    - 마지막으로 proxy는 모든 shard result들을 모아 최종 결과로 병합하고 client에 반환한다.







## 설치

- Milvus deployment options
  - Milvus lite
    - Milvus의 경량화 버전으로, Python package 형태로 사용이 가능하다.
    - 제한된 자원을 가진 기기에서 사용하거나, 빠르게 프로토타입을 만들어야 할 때 유용하게 사용할 수 있다.
    - 수백만개의 vector까지는 milvus lite로 충분히 작업이 가능하다.
  -  Milvus standalone
    - 단일 server에 사용해야 할 때 선택할 수 있는 옵션이다.
    - Docker, Docker Compose등을 통해 설치가 가능하다.
  - Milvus distributed
    - 여러 server에서 Milvus를 사용해야 할 경우 선택해야 하는 옵션이다.
    - Kubernetes cluster를 통해 설치가 가능하다.



- Milvus lite 설치

  - Python package를 설치한다.

  ```bash
  $ pip install -U pymilvus[milvus-lite]
  ```

  - 아래와 같이 사용한다.

  ```python
  from pymilvus import MilvusClient
  client = MilvusClient("./milvus_demo.db")
  ```



- Milvus standalone 설치

  - docker-compose.yml

  ```yaml
  version: '3.5'
  
  services:
    etcd:
      container_name: milvus-etcd
      image: quay.io/coreos/etcd:v3.5.18
      environment:
        - ETCD_AUTO_COMPACTION_MODE=revision
        - ETCD_AUTO_COMPACTION_RETENTION=1000
        - ETCD_QUOTA_BACKEND_BYTES=4294967296
        - ETCD_SNAPSHOT_COUNT=50000
      volumes:
        - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/etcd:/etcd
      command: etcd -advertise-client-urls=http://etcd:2379 -listen-client-urls http://0.0.0.0:2379 --data-dir /etcd
      healthcheck:
        test: ["CMD", "etcdctl", "endpoint", "health"]
        interval: 30s
        timeout: 20s
        retries: 3
  
    minio:
      container_name: milvus-minio
      image: minio/minio:RELEASE.2024-12-18T13-15-44Z
      environment:
        MINIO_ACCESS_KEY: minioadmin
        MINIO_SECRET_KEY: minioadmin
      ports:
        - "9001:9001"
        - "9000:9000"
      volumes:
        - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/minio:/minio_data
      command: minio server /minio_data --console-address ":9001"
      healthcheck:
        test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
        interval: 30s
        timeout: 20s
        retries: 3
  
    standalone:
      container_name: milvus-standalone
      image: milvusdb/milvus:v2.6.4
      command: ["milvus", "run", "standalone"]
      security_opt:
      - seccomp:unconfined
      environment:
        ETCD_ENDPOINTS: etcd:2379
        MINIO_ADDRESS: minio:9000
        MQ_TYPE: woodpecker
      volumes:
        - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/milvus:/var/lib/milvus
      healthcheck:
        test: ["CMD", "curl", "-f", "http://localhost:9091/healthz"]
        interval: 30s
        start_period: 90s
        timeout: 20s
        retries: 3
      ports:
        - "19530:19530"
        - "9091:9091"
      depends_on:
        - "etcd"
        - "minio"
  
  networks:
    default:
      name: milvus
  ```

  - Container 실행하기

  ```bash
  $ docker compose up
  ```







## Database와 Collection

- Database

  - Data를 조직화하고 관리하기 위한 논리적 단위이다.
    - Collection보다 상위 개념이다.
    - 논리적으로 분리된 데이터들을 생성하여 multi-tenancy가 가능하다.
  - Database 생성하기

  ```python
  from pymilvus import MilvusClient
  
  client = MilvusClient(
      uri="http://localhost:19530",
      token="root:Milvus"
  )
  
  client.create_database(
      db_name="my_database_1"
  )
  ```

  - 아래와 같이 생성시에 속성 값을 설정하는 것도 가능하며, 설정 가능한 속성값들은 아래와 같다.
    - `database.replica.number`: replica의 개수.
    - `database.resource_groups`: resource group.
    - `database.diskQuota.mb`: DB가 최대로 사용할 수 있는 disk 공간.
    - `database.max.collections`: DB에 생성할 수 있는 collection의 최대 개수.
    - `database.force.deny.writing`: 쓰기 작업을 강제로 막을지 여부.
    - `database.force.deny.reading`: 읽기 작업을 강제로 막을지 여부.
    - `timezone`: timezone

  ```python
  client.create_database(
      db_name="my_database_2",
      properties={
          "database.replica.number": 3
      }
  )
  ```

  - Database 조회하기

  ```python
  # 모든 database 이름 조회
  client.list_databases()
  
  # 특정 database 상세 정보 조회
  client.describe_database(
      db_name="default"
  )
  ```

  - Database 속성 수정하기

  ```python
  client.alter_database_properties(
      db_name="my_database_1",
      properties={
          "database.max.collections": 10
      }
  )
  ```

  - Database 속성 삭제하기

  ```python
  client.drop_database_properties(
      db_name="my_database_1",
      property_keys=[
          "database.max.collections"
      ]
  )
  ```

  - Database switching하기

  ```python
  client.use_database(
      db_name="my_database_2"
  )
  ```

  - Database 삭제하기
    - default database는 삭제가 불가능하다.
    - DB를 삭제하기 전에 DB에 생성된 모든 collection들을 삭제해야 한다.

  ```python
  client.drop_database(
      db_name="my_database_2"
  )
  ```



- Collection
  - 하나의 DB에는 여러 개의 collection을 생성할 수 있다.
    - Collection에 저장되는 데이터를 entity라 부른다.
    - Collection은 RDB의 table과 유사하며, entity는 record와 유사하다.
    - Collection은 2차원 테이블로 구성된다.
    - 각각의 column은 field를 나타내고, 각 row는 entity를 나타낸다.
  - 하나의 collection은 하나 이상의 shard로 구성된다.
    - 기본적으로 모든 collection들은 하나의 shard를 가진다.
    - Collection을 생성할 때 shard의 개수를 지정할 수 있다.
  - Load와 Release
    - Collection을 load하는 것은 컬렉션에서 유사도 검색과 query를 수행하기 위한 선행 조건이다. 
    - Collection을 load하면, Milvus는 각 field의 모든 index 파일과 raw data를 memory에 적재해 검색과 query에 빠르게 응답할 수 있도록 한다.
    - Collection이 load된 이후에 추가된 entity들은 자동으로 load된다.
    - 다만 검색과 query는 메모리 사용량이 많은 작업이므로 비용을 절감하기 위해, 현재 사용하지 않는 컬렉션은 해제(release)할 것을 권장한다.

