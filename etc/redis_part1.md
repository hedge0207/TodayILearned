# Redis

- Redis(Remote Dictionary Server)
  - 키-값 기반의 인-메모리 데이터 저장소.
    - 인-메모리란 컴퓨터의 메인 메모리(RAM)에 DB 데이터와 같은 주요 데이터를 저장하는 것을 말한다.
    - 오픈소스이다.
  - 특징
    - 키-값 기반이기 때문에 키만 알고 있다면 바로 값에 접근할 수 있다.
    - 또한 디스크에 데이터를 쓰는 구조가 아니라 메모리에서 데이터를 처리하기 때문에 속도가 상당히 빠르다.
    - Single threaded로 한 번에 단 하나의 명령어만 실행할 수 있다.
    - 다양한 데이터 구조를 제공하여, 캐시 데이터 저장 외에도 인증 토큰 저장, Ranking Board 등으로도 주로 사용된다.
  - Redis는 굉장히 다양한 자료구조를 지원한다.
    - Strings: 단순한 키-값 매핑 구조.
    - Lists: 배열 형식의 데이터 구조, 처음과 끝에 데이터를 넣고 빼는 것은 속도가 빠르지만, 중간에 데이터를 삽입할 때는 어려움이 있다.
    - Sets: 순서가 없는 Strings 데이터 집합, 중복된 데이터는 하나로 처리하기 때문에, 중복에 대한 걱정을 하지 않아도 된다.
    - Sorted Sets: Sets과 같은 구조지만, Score를 통해 정렬이 가능하다.
    - Hashes: 키-값 구조를 여러개 가진 object 타입을 저장하기 좋은 구조이다.
    - Bitmaps: 0,1로 이루어진 구조.
    - Bit field 
    - Geospatial Index: 지구상 두 지점의 경도(longitude)와 위도(latitude)를 저장
    - Hyperloglogs: 집합의 카디널리티(원소의 갯수)를 추정하기 위한 데이터 구조.
    - Streams: 로그를 처리하는데 최적화된 자료 구조



- Redis의 메모리 관리
  - maxmemory
    - Redis의 메모리 한계 값을 설정한다.
  - maxmemory 초과로 인해서 데이터가 지워지게 되는 것을 eviction이라고 한다.
    - Redis에 들어가서 INFO 명령어를 친 후 `eviced_keys` 수치를 보면 eviction이 왜 발생했는지 알 수 있다.
    - Amazon Elasticache를 사용하는 경우 mornitoring tab에 들어가면 eviction에 대한 그래프가 있는데, 이를 통해 Eviction 여부에 대한 알람을 받는 것이 가능하다.
  - maxmemory-policy
    - maxmemory에서 설정해준 수치까지 메모리가 다 차는 경우 추가 메모리를 확보하는 방식에 관한 설정.
    - noeviction: 기존 데이터를 삭제하지 않는다. 메모리가 꽉 차면 OOM 에러를 반환하고 새로운 데이터는 버린다.
    - allkeys-lru: LRU(Least Recently Used)라는 페이지 교체 알고리즘을 통해 데이터를 삭제하여 공간을 확보한다.
    - volatile-random: expire set을 가진 것 중 랜덤으로 데이터를 삭제하여 공간을 확보한다.
    - volatile-ttl: expire set을 가진 것 중 TTL(Time To Live) 값이 짧은 것부터 삭제한다.
    - allkeys-lfu: 가장 적게 액세스한 키를 제거하여 공간을 확보한다.
    - volate-lfu: expire set을 가진 것 중 가장 적게 액세스한 키부터 제거하여 공간을 확보한다.
  - COW(Copy On Write)
    - 쓰기 요청이 오면 OS는 `fork()`를 통해서 자식 프로세스를 생성한다.
    - `fork()`시에는 다른 가상 메모리 주소를 할당받지만 물리 메모리 블록을 공유한다.
    - 쓰기 작업을 시작하는 순간에는 수정할 메모리 페이지를 복사한 후에 쓰기 작업을 진행한다.
    - 즉 기존에 쓰던 메모리보다 추가적인 메모리가 필요하다.
    - 전체 페이지 중에서 얼마나 작업이 진행될지를 모르기 때문에 fork시에는 기본적으로 복사할 사이즈만큼의 free memory가 필요하다.
    - Redis를 직접 설치할 때 `/proc/sys/vm/overcommit_memory` 값을 1로 설정하지 않아 장애가 발생할 때가 있는데, `overcommit_memory`가 0이면 OS는 주어진 메모리량보다 크게 할당할 수가 없다.
    - 즉 `fork()`시에 충분한 메모리가 없다고 판단하여 에러를 발생시킨다.
    - 따라서 일단은 1로 설정해서 OS가 over해서 메모리를 할당할 수 있도록 한 후 maxmemory에 도달한 경우 policy에 따라 처리되도록 설정하는 것이 좋다.
  - used_memory_rss
    - RSS값은 데이터를 포함해서 실제로 redis가 사용하고 있는 메모리인데, 이 값은 실제로 사용하고 있는 used_memory 값보다 클 수 있다.
    - OS가 메모리를 할당할 때 page 사이즈의 배수만큼 할당하기 때문이다.
    - 이를 파편화(Fragmentations) 현상이라고 한다.



- Redis Replication
  - Redis를 구성하는 방법 중 Read 분산과 데이터 이중화를 위한 Master/Slave 구조가 있다.
    - Master 노드는 쓰기/읽기를 전부 수행하고, Slave는 읽기만 가능하다.
    - 이렇게 하려면 Slave는 Master의 데이터를 전부 가지고 있어야 한다.
    - Replication은 마스터에 있는 데이터를 복제해서 Slave로 옮기는 작업이다.
  - Master-Slave간 Replication 작업 순서
    - Slave Configuration 쪽에 `replicaof <master IP> <master PORT>` 설정을 하거나 REPLCAOF 명령어를 통해 마스터에 데이터 Sync를 요청한다.
    - Master는 백그라운드에서 RDB(현재 메모리 상태를 담은 파일) 파일 생성을 위한 프로세스를 진행하는데, 이 때 Master는 fork를 통해 메모리를 복사한다. 이후에 fork한 프로세스에서 현재 메모리 정보를 디스크에 덤프뜨는 작업을 진행한다.
    - Master가  fork할 때 자신이 쓰고 있는 메모리 만큼 추가로 필요해지므로 OOM이 발생하지 않도록 주의해야 한다.
    - 위 작업과 동시에 Master는 이후부터 들어오는 쓰기 명령들을 Buffer에 저장해 놓는다.
    - 덤프 작업이 완료되면 Master는 Slave에 해당 RDB 파일을 전달해주고, Slave는 디스크에 저장한 후에 메모리로 로드한다.
    - Buffer에 모아두었던 쓰기 명령들을 전부 slave로 보내준다.



- Redis Cluster
  - 여러 노드가 hash 기반의 slot을 나눠 가지면서 클러스터를 구성하여 사용하는 방식.
  - 전체 slot은 16384이며, hash 알고리즘은 CRC16을 사용한다.
    - Key를 CRC16으로 해시한 후에 이를 16384로 나누면 해당 key가 저장될 slot이 결정된다.
  - 클러스터를 구성하는 각 노드들은 master 노드로, 자신만의 특정 slot range를 갖는다.
    - 데이터를 이중화하기 위해 slave 노드를 가질 수 있다.
    - 만약 특정 master 노드가 죽게 되면, 해당 노드의 slave 중 하나가 master로 승격하여 역할을 수행하게 된다.
    - 하나의 master node는 여러 개의 slave node를 가질 수 있다.





# 구성

- 레디스는 단일 인스턴스만으로도 충분히 운영이 가능하다.
  - 그러나 아래와 같은 경우 다른 운영 방식을 도입할 필요가 있다.
    - 물리 머신이 가진 메모리의 한계를 초과하는 데이터 저장
    - Failover에 대한 처리를 통해 HA(High Availability, 고가용성)을 보장
  - Master가 죽을 경우 Slave는 Sync 에러를 발생시킨다.
    - 이 상태에서는 쓰기는 불가능하고 읽기만 가능하다.
    - 따라서 Slave를 Master로 승격시켜야 한다.
  - 다음과 같은 운영 방식이 존재한다.
    - Replication
    - 센티넬(Sentienl)
    - 클러스터(Cluster)



- Replication

  - Master - Replica 형태의 복제를 제공한다.
    - Master-Slave 혹은, Leader-Follower 라고도 한다.
    - 한 개의 마스터 노드에 여러 개의 레플리카 노드가 있을 수 있고, 레플리카 노드에 또 다른 레플리카 노드가 연결되는 것도 가능하다.
    - 단, 한 개의 복제 그룹에서는 항상 한 개의 마스터 노드만 존재한다.
    - 복제가 연결되어 있는 동안 마스터 노드의 데이터는 실시간으로 레플리카 노드에 복사된다.
    - 따라서 서비스를 제공하던 마스터 노드가 다운되더라도 레플리카 노드에 어플리케이션을 재연결 해주면 서비스를 계속 할 수 있다.
    - Master는 읽기와 쓰기를 담당하며 Slave는 읽기만을 담당한다.
    - Slave에 쓰기도 가능하게 설정할 수 있지만 성능상 효율이 매우 떨어진다.

  - 복제하기

  ```bash
  $ replicaof <master의 host> <master의 port>
  ```

  - 과정
    - `replicaof` 명령어를 받은 마스터 노드는 자식 프로세스를 생성하여 백그라운드로 덤프 파일을 만든다.
    - 이를 네트워크를 통해 레플리카 노드에 보낸다.
    - 이 파일을 받은 레플리카 노드는 데이터를 메모리로 로드한다.
  - 마스터까지만 데이터가 입력된 후 마스터 노드가 죽는다면 이 데이터는 레플리카까지 전달되지 않았기에 유실이 발생할 수 있다.
    - 현재는 이 현상을 디버깅하기 힘들 정도로 데이터의 전달 속도가 매우 빠르기 때문에 유실이 자주 발생되지는 않을 것이다.
  - 이 방식은 고가용성을 보장하지는 않는다.
    - 이 구성 만으로는 자동 장애 감지 및 복구가 불가능하다.
    - 이 구성의 목적은 읽기 부하의 분산에 있지 고가용성의 확보에 있지 않다.



- Sentinel
  - 레디스 프로세스가 다운되면 메모리 내에 저장됐던 데이터는 유실된다.
    - 마스터 노드에 연결된 복제 노드가 있어도, 운영 중인 서비스에서 어플리케이션이 마스터에 연결된 상태였다면 다음 과정을 직접 수행해야 한다.
    - 복제 노드에 접속해서 `REPLICAOF NO ONE` 커맨드를 통해 마스터 연결 해제
    - 어플리케이션 코드에서 레디스 연결 설정을 변경(마스터 노드의 IP에서 복제 노드의 IP로)
    - 배포
  - Sentinel은 위와 같은 과정을 자동으로 수행해준다.
    - 마스터 노드와 복제 노드를 계속 모니터링하면서 장애 상황이 발생하면 복제 노드를 마스터로 승격시키기 위해 자동 failover를 진행한다.
    - failover: 시스템 장애시 대체본으로 자동으로 변경되어서 장애를 극복하는 것
    - 담당자에게 메일을 보내도록 알림을 설정하는 것도 가능하다.
  - Sentinel 인스턴스
    - 정상적인 기능을 위해서는 적어도 3개의 Sentinel 인스턴스가 필요하다(3개 이상의 홀수 인스턴스가 필요하다).
    - 각 Sentinel 인스턴스는 레디스의 모든 노드를 감시하며 서로 연결되어 있다.
    - 과반수 이상이 동의해야만 failover를 시작할 수 있다.
    - 어플리케이션은 마스터나 복제 노드에 직접 연결하지 않고, Sentinel 노드와 연결한다.
    - Sentinel 노드는 어플리케이션에게 현재 마스터의 IP. Port를 알려주며, failover 이후에는 새로운 마스터의 IP, Port 정보를 알려준다.
  - Failover 과정
    - 마스터가 다운되고, 이를 감시하고 있던 Sentinel은 마스터에 진짜 연결할 수 없는지에 대한 투표를 시작한다.
    - 과반수 이상이 동의하면 failover를 시작한다.
    - 연결이 안되는 마스터에 대한 복제 연결을 끊고, 복제 노드 중 한 개를 마스터로 승격시킨다.
    - 만일 다른 복제 노드가 있다면 해당 노드를 새로 승격된 마스터 노드에 연결시킨다.
    - 만일 다운되었던 기존 마스터 노드가 다시 살아나면 새로운 마스터에 복제본으로 연결된다.



- Cluster

  - 위에서 살펴본 모든 방식에 샤딩 기능까지 더해진 운영 방식
    - 확장성, 고성능, 고가용성이 특징이다.
  - 구성
    - 모든 노드는 서로 연결된 Full Mesh 구조를 이루고 있다.
    - 모든 마스터와 복제 노드는 서로 연결되어 있으며, 가십 프로토콜을 이용해 통신한다.
    - 클러스터를 사용하기 위해서는 최소 세 개의 마스터 노드가 필요하다.
  - 샤딩
    - 어플리케이션으로부터 들어오는 모든 데이터는 해시 슬롯에 저장된다.
    - 레디스 클러스터는 총 16384개의 슬롯을 가지며, 마스터 노드는 슬롯을 나누어 저장한다.
    - 예를 들어 노드가 2개인 경우 노드  A는 0~16384//2까지, 노드B는 16384//2+1~ 16834까지 저장하는 방식이다.
    - 입력되는 모든 키는 CRC16 알고리즘을 사용하여 슬롯에 매핑된다.
    - 해시 슬롯은 마스터 노드 내에서 자유롭게 옮겨질 수 있으며, 이를 위한 다운타임은 필요하지 않다.

  - Client Redirection
    - 어플리케이션은 아무 노드에나 쿼리를 보낼 수 있다.
    - 단, 요청한 키가 쿼리를 보낸 노드에 존재하지 않을 수 있는데, 이 때 해당 노드는 요청한 키가 존재하는 노드 정보를 반환한다.
    - 이를 리다이렉트 메세지라 한다.

  - failover
    - Sentinel 구조와 마찬가지로 자동으로 failover가 발생한다.
    - Sentinel 구조에서는 Sentinel 프로세스가 노드들을 감시했지만, 레디스 클러스터 구조에서는 모든 노드가 서로 감시한다는 점에 차이가 있다.





# 설치

- Docker로 Redis 설치하기

  - Redis 이미지 받기

  ```bash
  $ docker pull redis[:원하는 버전]
  ```

  - Docker network 생성하기
    - redis와 redis-cli 사이의 통신에 필요하다.
    - redis-cli를 사용하지 않을 경우 생성하지 않아도 된다.

  ```bash
  $ docker network create <network 이름>
  ```

  - Docker 컨테이너 생성 및 실행
    - 기본 port는 6379이다.

  ```bash
  $ docker run --network <network 이름> -p <port 번호>:<port 번호> redis
  ```



- 레디스 설치 후 꼭 확인해야 할 파라미터들

  - 보안 관련 파라미터
    - `protected-mode`는 yes로 준다(기본값).
    - `requirepass`에 접속할 때 사용할 패스워드를 입력한다.

  ```yaml
  bind <서버 IP>
  protected-mode yes
  requirepass <"password">
  ```

  - persistence 관련 파라미터
    - `save`는 time 내에 count 만큼의 key 변경이 발생하면 dump.rdb 파일을 작성한다(사용하지 않을 경우 `""`로 주면 된다).
    - `save`는 여러 개 지정할 수 있으며 or 연산이므로 지정한 조건들 중 하나라도 일치하면 저장된다.
    - `stop-writes-on-bgsave-error`가 yes인 경우 RDB 파일을 작성하다 실패하면 이후 모든 쓰기 요청을 막는다(SAVE 명령에만 적용되며 BGSAVE에는 적용되지 않는다).
    - `appendonly`: AOF 기능을 사용할지 여부를 설정한다.
    - `appendfsync`: AOF 파일에 기록하는 시점을 설정한다.
  - auto-aof-rewrite-percentage: AOF 파일 사이즈가 숫자값 % 이상으로 커지면 rewrite한다(%의 기준은 레디스 서버가 시작할 시점의 AOF 파일 사이즈이다).

  ```txt
  # RDB
  save [time], [count]
  stop-writes-on-bgsave-error <yes/no>
  
  # AOF
  appendonly <yes/no>
  appendfsync [always/everysec/no]
  auto-aof-rewrite-percentage [0-100]
  ```

  - replica 관련 파라미터들

  ```bash
  replicaof <마스터 IP> <마스터 port>
  masterauth <"마스터의 requirepass"
  ```

    - back/foreground 실행 설정
      - no로 설정할 경우(default) foregrund로 실행된다.

  ```bash
  daemonize [yes/no]
  ```

    - 메모리 관련 설정
      - 64bit 환경에서는 maxmemory의 default 값은 0이며, swap 메모리를 사용할 때까지 계속해서 커지게 된다.
      - 32bit 환경에서는 3GB가 기본 값이다.

  ```bash
  maxmemory <가용 메모리의 60~70%>
  maxmemory-policy <policy>
  ```





## Replication으로 설치

- 2 대의 서버에 master-slave 구조로 설치한다.

  - 설치를 위해 Redis Docker image를 준비한다.

  ```bash
  $ docker pull redis:6.2.6
  ```

  - 앞서 말했듯 이 구조는 고가용성을 지원하지는 않는다.



- 설치하기

  - A 서버에 아래와 같이 docker-compose file을 작성한다.

  ```yaml
  services:
    redis-master:
      image: redis:6.2.6
      container_name: redis-master
      ports:
        - 6379:6379
      command: redis-server --appendonly yes
  ```

  - Redis container를 실행한다.

  ```bash
  $ docker compose up
  ```

  - B서버에 아래와 같이 docker-compose file을 작성한다.

  ```yaml
  services:
    redis-slave:
      image: redis:6.2.6
      container_name: redis-slave
      ports:
        - 6379:6379
      command: redis-server --replicaof <serverA_host> 6379
  ```

  - Redis container를 실행한다.

  ```bash
  $ docker compose up
  ```



- 테스트

  - Python Redis client를 사용하여 master와 slave 모두에 연결해본다.

  ```python
  from redis import Redis
  
  master_redis = Redis(host="<serverA_host>")
  slave_redis = Redis(host="<serverB_host>")
  
  # 둘 모두 연결은 가능하다.
  print(master_redis.ping())		# True
  print(slave_redis.ping())		# True
  ```

  - 다만 slave node의 경우 쓰기는 불가능하고 읽기만 가능하다.

  ```python
  from redis.exceptions import ReadOnlyError
  
  
  master_redis.set("foo", "bar")
  try:
      slave_redis.set("baz", "qux")
  except ReadOnlyError:
      print("Error!")			# Error!
  
  master_redis.get("foo")		# b'bar'
  slave_redis.get("bar")		# b'bar'
  ```







## Sentinel로 설치

- Sentinel로 사용할 node에 설정해줘야 하는 값들은 아래와 같다.

  - `port`
    - Sentinel node가 리스닝할 포트 번호를 설정한다.
    - 기본값은 26379
  - `sentinel monitor <name> <ip> <port> <quorum>`
    - Sentinel node가 감시할 master node의 정보를 입력한다.
    - `<ip> <port>`에 해당하는 Redis master node를 `<name>`이라는 이름으로 등록하겠다는 의미이며, 이후 다른 설정에서 `<name>`에 설정한 값을 사용한다.
    - IP 대신 hostname도 사용은 가능하다.
    - `quorum`은 failover를 위해 필요한 sentinel의 수이며, 모든 sentinel에 동일한 값을 입력해야한다. 
    - 여기서 설정한 master node와 연결된 slave node에도 자동으로 연결된다.
    - 필수 설정이므로 기본값은 없다.

  - `sentinel down-after-milliseconds <name> <milliseconds>`
    - Sentinel과 연결된 master 혹은 slave를 다운으로 간주하기까지 대기할 시간을 설정한다.
    - 기본값은 30000(30초)이다.
    - 너무 짧을 경우 정상 노드를 비정상이라고 판단할 수 있다.
  - `sentinel failover-timeout <name> <milliseconds>`
    - Failover과정에 허용되는 시간이다.
    - 이 시간 내에 slave가 master로 승격되지 못하면 해당 slave는 master 후보에서 제외된다.
    - 기본값은 180000이다.
  - `sentinel parallel-syncs <name> <count>`
    - Failover이후 새로운 마스터로부터 슬레이브들이 동시에 동기화할 수 있는 최대 수
    - 슬레이브 수가 많을 경우, 이 값을 늘리면 빠르게 클러스터가 안정화되지만, 너무 높이면 마스터 과부하될 가능성이 있다.
    - 기본값은 1이다.

  - 예시

  ```
  port 26379
  
  sentinel monitor mymaster 172.28.0.10 6379 2
  sentinel down-after-milliseconds mymaster 5000
  sentinel failover-timeout mymaster 10000
  sentinel parallel-syncs mymaster 1
  ```



- 설치하기

  - Sentinel 설정 파일 준비
    - 세 개의 sentinel 모두 동일한 설정 파일을 사용한다.

  ```
  port 26379
  
  sentinel monitor mymaster 172.28.0.10 6379 2
  sentinel down-after-milliseconds mymaster 5000
  sentinel failover-timeout mymaster 10000
  sentinel parallel-syncs mymaster 1
  ```

  - docker-compose.yml
    - `redis-net`에 subnet을 지정하고, 각 컨테이너별로 `ipv4_address`를 지정한 이유는 `sentinel monitor <name> <ip> <port> <quorum>`를 설정할 때 `<ip>`에 hostname을 설정할 경우 에러가 발생하면서 sentinel node의 실행이 실패했기 때문이다.
    - Redis 버전이 낮아서 발생하는 문제인지는 확인이 필요하다.

  ```yaml
  services:
  
    redis-node-1:
      image: redis:6.2.6
      container_name: redis-node-1
      ports:
        - "6379:6379"
      command: redis-server --appendonly yes
      networks:
        redis-net:
          ipv4_address: 172.28.0.10
  
    sentinel-node-1:
      image: redis:6.2.6
      container_name: sentinel-node-1
      depends_on:
        - redis-node-1
      ports:
        - "26379:26379"
      command: redis-sentinel /sentinel/sentinel.conf
      volumes:
        - ./sentinel-1.conf:/sentinel/sentinel.conf
      networks:
        redis-net:
          ipv4_address: 172.28.0.11
  
    redis-node-2:
      image: redis:6.2.6
      container_name: redis-node-2
      ports:
        - "6380:6379"
      command: redis-server --replicaof redis-node-1 6379 --appendonly yes
      networks:
        redis-net:
          ipv4_address: 172.28.0.12
  
    sentinel-node-2:
      image: redis:6.2.6
      container_name: sentinel-node-2
      depends_on:
        - redis-node-2
      ports:
        - "26380:26379"
      command: >
        redis-sentinel /sentinel/sentinel.conf
      volumes:
        - ./sentinel-2.conf:/sentinel/sentinel.conf
      networks:
        redis-net:
          ipv4_address: 172.28.0.13
  
    redis-node-3:
      image: redis:6.2.6
      container_name: redis-node-3
      ports:
        - "6381:6379"
      command: redis-server --replicaof redis-node-1 6379 --appendonly yes
      networks:
        redis-net:
          ipv4_address: 172.28.0.14
  
    sentinel-node-3:
      image: redis:6.2.6
      container_name: sentinel-node-3
      depends_on:
        - redis-node-3
      ports:
        - "26381:26379"
      command: redis-sentinel /sentinel/sentinel.conf
      volumes:
        - ./sentinel-3.conf:/sentinel/sentinel.conf
      networks:
        redis-net:
          ipv4_address: 172.28.0.15
  
  networks:
    redis-net:
      driver: bridge
      ipam:
        config:
          - subnet: 172.28.0.0/16
  ```





## Cluster로 설치

- Clustering 관련 설정은 아래와 같다.

  - `cluster-enabled`
    - Cluster mode로 실행할지 여부를 설정한다.
    - `yes`로 줄 경우 cluster mode로 실행된다.
  - `cluster-config-file`
    - 클러스터의 상태를 기록하는 파일의 위치를 설정한다.
    - 클러스터의 상태가 변경될 때 마다 설정된 경로의 파일에 변경 사항이 기록된다.
    - `redis` docker image 기준으로 `/data`를 기준으로 경로가 설정된다.
  - `cluster-node-timeout`
    - Cluster를 구성 중인 node가 down되었다고 판단하는 시간이다.
    - 단위는 millisecond이다.
  - 예시

  ```toml
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 3000
  ```

  - Port
    - 기본 포트에 10000을 더한 포트를 클러스터 버스 포트로 사용한다.
    - 따라서 기본 포트와 기본 포트에 10000을 더한 포트 모두 사용 가능해야 한다.



- 각 redis node 별로 설정 파일을 준비한다.

  - `redis1.conf`

  ```toml
  port 7001
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 3000
  appendonly yes
  ```

  - `redis2.conf`

  ```toml
  port 7002
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 3000
  appendonly yes
  ```

  - `redis3.conf`

  ```toml
  port 7003
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 3000
  appendonly yes
  ```



- Docker compose로 실행하기

  - 아래와 같이 docker compose file을 작성한다.

  ```yaml
  services:
    redis1:
      image: redis:6.2.6
      container_name: redis1
      environment:
        - TZ=Asia/Seoul
      volumes:
        - ./config/redis1.conf:/etc/redis.conf
      command: redis-server /etc/redis.conf
      restart: always
      ports:
        - 7001:7001
        - 17001:17001
    
    redis2:
      image: redis:6.2.6
      container_name: redis2
      environment:
        - TZ=Asia/Seoul
      volumes:
        - ./config/redis2.conf:/etc/redis.conf
      command: redis-server /etc/redis.conf
      restart: always
      ports:
        - 7002:7002
        - 17002:17002
    
    redis3:
      image: redis:6.2.6
      container_name: redis3
      environment:
        - TZ=Asia/Seoul
      volumes:
        - ./config/redis3.conf:/etc/redis.conf
      command: redis-server /etc/redis.conf
      restart: always
      ports:
        - 7003:7003
        - 17003:17003
  ```

  - 실행하기

  ```bash
  $ docker compose up
  ```



- 클러스터 설정하기

  - Docker container 내부에 attach한다.

  ```bash
  $ docker exec -it redis1 /bin/bash
  ```

  - redis-cli를 사용하여 cluster를 구성한다.

  ```bash
  $ redis-cli --cluster create <host>:7001 <host>:7002 <host>:7003
  ```

  - 정상적으로 구성 됐는지 확인한다.

  ```bash
  $ redis-cli -p 7001
  cluster nodes
  cluster info
  ```



- Slave 추가하기

  - 위 구조에서는 slave를 설정하지 않았기 때문에, 고가용성이 보장되지 않는다.
    - 고가용성이 보장되도록 하기 위해서는 아래와 같이 slave node를 함께 띄워야 한다.

  - Slave node들을 위한 설정 파일을 작성한다.

  ```toml
  # redis-slave1.conf
  port 7101
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 3000
  appendonly yes
  
  # redis-slave2.conf
  port 7102
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 3000
  appendonly yes
  
  # redis-slave3.conf
  port 7103
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 3000
  appendonly yes
  ```

  - Docker compose file에 slave 노드을 추가한다.

  ```yaml
  services:
    redis1:
      image: redis:6.2.6
      container_name: redis1
      environment:
        - TZ=Asia/Seoul
      volumes:
        - ./config/redis1.conf:/etc/redis.conf
      command: redis-server /etc/redis.conf
      restart: always
      ports:
        - 7001:7001
        - 17001:17001
    
    redis2:
      image: redis:6.2.6
      container_name: redis2
      environment:
        - TZ=Asia/Seoul
      volumes:
        - ./config/redis2.conf:/etc/redis.conf
      command: redis-server /etc/redis.conf
      restart: always
      ports:
        - 7002:7002
        - 17002:17002
    
    redis3:
      image: redis:6.2.6
      container_name: redis3
      environment:
        - TZ=Asia/Seoul
      volumes:
        - ./config/redis3.conf:/etc/redis.conf
      command: redis-server /etc/redis.conf
      restart: always
      ports:
        - 7003:7003
        - 17003:17003
    
    redis-slave1:
      image: redis:6.2.6
      container_name: redis-slave1
      environment:
        - TZ=Asia/Seoul
      volumes:
        - ./config/redis-slave1.conf:/etc/redis.conf
      command: redis-server /etc/redis.conf
      restart: always
      ports:
        - 7101:7101
        - 17101:17101
      
    redis-slave2:
      image: redis:6.2.6
      container_name: redis-slave2
      environment:
        - TZ=Asia/Seoul
      volumes:
        - ./config/redis-slave2.conf:/etc/redis.conf
      command: redis-server /etc/redis.conf
      restart: always
      ports:
        - 7102:7102
        - 17102:17102
      
    redis-slave3:
      image: redis:6.2.6
      container_name: redis-slave3
      environment:
        - TZ=Asia/Seoul
      volumes:
        - ./config/redis-slave3.conf:/etc/redis.conf
      command: redis-server /etc/redis.conf
      restart: always
      ports:
        - 7103:7103
        - 17103:17103
  ```

  - Cluster를 설정한다.

  ```bash
  $ docker exec -it redis1 /bin/bash
  $ redis-cli --cluster create redis1:7001 redis2:7002 redis3:7003
  $ redis-cli -p 7001
  cluster info
  cluster nodes
  ```

  - Slave를 추가한다.
    - `add-node`가 클러스터에 새로운 node를 추가하는 명령어이며, `--cluster-slave`는 slave node로 추가한다는 옵션이다.
    - 만약 `--cluster-slave`를 주지 않을 경우 master로 추가된다.
  
  
  ```bash
  $ redis-cli --cluster add-node redis-slave1:7101 redis1:7001 --cluster-slave
  $ redis-cli --cluster add-node redis-slave2:7102 redis2:7002 --cluster-slave
  $ redis-cli --cluster add-node redis-slave3:7103 redis3:7003 --cluster-slave
  ```
  
  - 정상적으로 추가 됐는지 확인한다.
  
  ```bash
  $ redis-cli -p 7001
  cluster info
  cluster nodes 
  ```
  
  - 아래와 같이 cluster 생성과 동시에 slave를 설정하는 것도 가능하다.
  
  ```bash
  $ redis-cli --cluster create \
    redis1:7001 \
    redis2:7002 \
    redis3:7003 \
    redis-slave1:7101 \
    redis-slave2:7102 \
    redis-slave3:7103 \
    --cluster-replicas 1
  ```



- Cluster가 생성되는 과정

  - Cluster에 참여하는 노드들이 실행된다.
    - `cluster-enabled yes`값이 설정된 노드들은 cluster에 참여하는 노드가 된다.
  - Cluster 생성 명령어가 실행된다.
    - 아래 명령어가 실행되면 명령어를 실행한 노드가 각 노드에게 `CLUSTER MEET` 메시지를 보낸다.

  ```bash
  $ redis-cli --cluster create <host>:<port> [...] [--cluster-replicas <num>]
  ```

  - Cluster에 참여하려는 노드들은 `CLUSTER MEET` 메시지를 받은 후 자신을 광고(advertise)한다.
    - 이 때, 자신의 IP와 port를 광고하는데, `cluster-announce-ip`, `cluster-announce-port`에 설정된 IP와 port로 광고한다.
    -  `cluster-announce-ip`의 기본값은 eth0 같은 인터페이스에서 자신의 IP를 추론한다. 위 예시의 경우 그 결과로 docker container가 자신이 속한 docker network에서 할당 받은 IP값이 설정된다.
    - `cluster-announce-port`의 기본 값은 `redis.conf`에 설정해준 `port` 값이다.
    - 다른 노드들은 이 정보를 기반으로 해당 노드에 연결을 시도한다.
  - 각 노드가 서로 연결되면 master/slave 역할을 나눈다.



- Redis cluster에서 노드 제거하기

  - `del-node` 명령어를 사용한다.

  ```bash
  $ redis-cli --cluster del-node <명령어를 수행할 노드의 IP>:<명령어를 수행할 노드의 port> <삭제할 노드의 ID>
  
  # e.g. serverA에서 6380 port로 실행중인 node에서 ID가 a1a4ae72d5719c2fe9c8504a02e425324646346인 node를 cluster에서 제거하는 작업을 하는 경우 아래와 같이 하면 된다.
  $ redis-cli --cluster del node <serverA_IP>:6380 a1a4ae72d5719c2fe9c8504a02e425324646346
  ```

  - 만약 제거하려는 node가 master인 경우 아래와 같이 reshard를 먼저 수행해야한다.
    - Slave일 경우 하지 않아도 된다.

  ```bash
  $ redis-cli --cluster reshard <reshard를 수행될 노드의 IP>:<reshard가 수행될 노드의 port>
  
  # e.g. 만약 삭제를 위해 reshard를 실행하고자 하는 master node의 IP가 11.22.33.44이고, port가 6380이라면 아래와 같이 하면 된다.
  $ redis-cli --cluster reshard 11.22.33.44:6380
  ```

  



- Cluster를 구성한 후 slave node에서 아래와 같은 메시지가 반복적으로 출력되는 경우의 해결 방법

  - Slave node에서 아래와 같은 메시지가 반복적으로 출력된다면

  ```
  Retrying with SYNC...
  MASTER aborted replication with an error: NOAUTH Authentication required.
  Reconnecting to MASTER 172.21.0.4:6380 after failure
  MASTER <-> REPLICA sync started
  Non blocking connect for SYNC fired the event.
  Master replied to PING, replication can continue...
  (Non critical) Master does not understand REPLCONF listening-port: -NOAUTH Authentication required.
  (Non critical) Master does not understand REPLCONF capa: -NOAUTH Authentication required.
  Partial resynchronization not possible (no cached master)
  Unexpected reply to PSYNC from master: -NOAUTH Authentication required.
  ```

  - Master node의 설정에 `requirepass`를 설정했을 것이다.
    - 이는 Redis에 접근할 때 사용하는 비밀번호를 설정하는 것이다.
    - Master와 slave 사이의 통신에도 이 값을 사용하는데, slave가 master와 통신할 때 이 값을 알 수 있게 해줘야한다.
    - 아래와 같이 `masterauth` 설정을 추가하고, master node의 `requirepass`에 설정한 값과 동일한 값을 설정한다.

  ```toml
  # master node의 requirepass 값이 "my_password"라면 slave node에 아래 설정을 추가한다.
  masterauth "my_password"
  ```



- Container가 삭제되었다가 재생성 되더라도 기존 cluster가 그대로 재구성되어야 하는 경우

  - Redis cluster에 속한 노드들은 `cluster-config-file`에 설정한 파일에 클러스터의 정보를 작성한다.
    - 따라서 container가 종료되더라도 이 파일만 유지되면 다시 container를 생성했을 때 기존과 동일한 클러스터를 구성할 수 있다.
  - Redis cluster 구성을 위해 아래와 같이 두 개의 설정 파일을 준비한다.
    - 세 대의 서버에서 모두 아래와 동일한 설정 파일을 `<server_ip>` 부분만 변경하여 작성한다.

  ```toml
  # redis1.conf
  protected-mode yes
  requirepass "my_password"
  masterauth "my_password"
  
  save 900 1
  
  appendonly yes
  appendfsync everysec
  
  maxmemory 30MB
  maxmemory-policy allkeys-lru
  
  port 6380
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 3000
  cluster-announce-ip <server_ip>
  
  
  
  # redis2.conf
  protected-mode yes
  requirepass "my_password"
  masterauth "my_password"
  
  save 900 1
  
  appendonly yes
  appendfsync everysec
  
  maxmemory 30MB
  maxmemory-policy allkeys-lru
  
  port 6381
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 3000
  cluster-announce-ip <server_ip>
  ```

  - 각 서버의 적절한 위치에 cluster의 정보가 저장될 nodes.conf 파일을 생성하고

  ```bash
  $ touch nodes1.conf nodes2.conf 
  ```

  - 아래와 같이 Redis node들을 생성한다.
    - 세 개의 서버에 모두 동일한 docker-compose.yml 파일을 작성한다.
    - 앞에서 생성한 `nodes1...2.conf` 파일과 container 내부의 `nodes.conf`파일을 바인딩한다.

  ```yaml
  services:
    redis1:
      image: redis:6.2.6
      container_name: redis1
      volumes:
        - ./redis1.conf:/etc/redis.conf
        - ./nodes1.conf:/data/nodes.conf
      command: redis-server /etc/redis.conf
      restart: always
      ports:
        - 6380:6380
        - 16380:16380
    
    redis2:
      image: redis:6.2.6
      container_name: redis2
      volumes:
        - ./redis2.conf:/etc/redis.conf
        - ./nodes2.conf:/data/nodes.conf
      command: redis-server /etc/redis.conf
      restart: always
      ports:
        - 6381:6381
        - 16381:16381
  ```

  - Cluster를 생성한다.

  ```bash
  $ docker exec -it redis1 bash
  
  $ redis-cli -a my_password --cluster create \
    <serverA_IP>:6380 \
    <serverA_IP>:6381 \
    <serverB_IP>:6380 \
    <serverB_IP>:6381 \
    <serverC_IP>:6380 \
    <serverC_IP>:6381 \
    --cluster-replicas 1
  ```

  - Cluster가 구성된 후 container를 삭제하고 다시 생성해도, cluster 정보가 유지된다.

  ```bash
  $ docker compose down
  $ docker compose up
  ```



- Docker를 사용하여 한 서버에 여러 대의 node를 띄워 cluseter를 구성할 때의 주의사항

  - 운영 환경이 아닌 테스트 환경에서는 한 서버에 여러 노드를 실행해 cluster를 구성해야 하는 경우가 있다.
    - Cluster 생성시에 위 예시에서와 같이 hostname을 사용하면 큰 문제가 없지만, IP를 사용하고자 할 경우 문제가 생길 수 있다.
    - 예를 들어 아래와 같이 cluster를 생성하려 할 경우, 아무리 기다려도 cluster가 생성되지 않는다.

  ```bash
  # host machine의 IP가 11.22.33.44라고 가정
  $ redis-cli --cluster create \
    11.22.33.44:7001 \
    11.22.33.44:7002 \
    11.22.33.44:7003 \
    11.22.33.44:7101 \
    11.22.33.44:7102 \
    11.22.33.44:7103 \
    --cluster-replicas 1
  ```

  - 원인
    - 잘못된 advertise로 인해 다른 노드에 연결되지 못 하기 때문이다.
    - 위 예시에서는 `redis.conf` 파일에 `cluster-announce-ip`값을 따로 설정해주지 않았다.
    - 따라서 redis가 이 값을 추론하여 container가 속한 docker network에서 할당 받은 IP값이 설정된다.
    - 즉, 각 노드는 `<container_IP>:<자신의 port>`로 자신을 광고한다.
    - 문제는 클러스터를 생성할 때는 host machine의 IP로 설정했으므로, 두 정보 사이에 불일치가 발생한다.
    - 이로 인해 각 노드들 사이의 연결은 실패하고 cluster는 구성되지 못한다.
  - 해결 방법1. `cluster-announce-ip`를 설정하면 해결이 가능하다.
    - `cluster-announce-ip`를 설정하여 각 노드가 정확한 IP로 자신을 advertise 할 수 있게 한다.

  ```yaml
  port 6379
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 3000
  cluster-announce-ip 11.22.33.44
  ```
  
  - 해결 방법2. cluster 생성시에 container IP를 입력한다.
    - 이 경우, container들이 재실행되면서 container IP가 변경되면 cluster 구성에 문제가 생길 것이 걱정될 수 있다.
    - 만약 재실행시에 기존과 동일한 IP를 부여 받은 node가 하나라도 있으면, `node.conf`와 node ID를 사용해서 자동으로 재구성된다(확인 필요).
  
  ```bash
  $ redis-cli --cluster create \
    <redis1_container_IP>:7001 \
    <redis2_container_IP>:7002 \
    # ...
    --cluster-replicas 1
  ```
  
  

