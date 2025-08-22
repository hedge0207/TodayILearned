# Metricbeat

- Metricbeat

  - Metric을 수집할 수 있도록 해주는 ELK Stack 중 하나이다.
  - module과 Ouput을 지정 가능하다.
    - module은 수집 할 metric의 종류, output에는 metric을 저장 할 곳을 지정하면 된다.
    - Elasticsearch Metric뿐 아니라 다양한 Metric을 수집할 수 있다.
  - 수집 할 수 있는 metric의 종류(module의 종류)

  > https://www.elastic.co/guide/en/beats/metricbeat/7.15/metricbeat-modules.html



- Metricbeat으로  Elasticsearch metric 수집하기(linux, docker 기준)

  - metricbeat 설치

  ```bash
  $ curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.14.0-linux-x86_64.tar.gz
  $ tar xzvf metricbeat-7.14.0-linux-x86_64.tar.gz
  ```

  - `metricbeat.yml` 파일 수정
    - 위에서 압축을 푼 폴더 내부에 있는 `metricbeat.yml` 파일을 아래와 같이 수정한다.

  ```yaml
  # (...)
  # ================================== Outputs ===================================
  
  # Configure what output to use when sending the data collected by the beat.
  
  # ---------------------------- Elasticsearch Output ----------------------------
  output.elasticsearch:
    # Array of hosts to connect to.
    hosts: ["localhost:9200"]
    # (...)
  setup.kibana:
    host: "localhost:9200"
  # (...)
  ```

  - elasticsearch module을 활성화한다.

  ```bash
  $ ./metricbeat modules enable elasticsearch
  ```

  - 활성화된 module list 보기

  ```bash
  $ ./metricbeat modules list
  ```

  - `modules.d` 폴더 내부의 `elasticsearch.yml` 파일을 아래와 같이 수정한다.
    - `modules.d` 폴더에는 각 모듈의 설정 파일이 저장되어 있다.
    - 활성화 되지 않은 모듈의 설정 파일은 suffix로 `.disable`가 붙어있다.
    - `metricsets`에 입력된 값들을 수집하겠다는 의미이며, 전부 수집하지 않아도 된다면, 필요한 것만 입력하면 된다.

  ```yaml
  - module: elasticsearch
    metricsets:
      - ccr
      - cluster_stats
      - enrich
      - node
      - node_stats
      - index
      - index_recovery
      - index_summary
      - shard
      - ml_job
      - pending_tasks
    period: 10s
    hosts: ["http://localhost:9200"]
    #username: "user"
    #password: "secret"
  ```

  - ES에서 수집 가능한 metric 목록

  > https://www.elastic.co/guide/en/beats/metricbeat/7.15/metricbeat-module-elasticsearch.html

  - setup하기
    - `metricbeat.yml`에서 `setup.kibana`에 설정해준대로 setup된다.


  ```bash
  $ ./metricbeat setup
  ```

  - 실행하기
    - `-e` 옵션을 주면 log를 터미널에 출력해준다.
    - 주지 않을 경우 `/usr/share/metricbeat/logs`에 log 파일이 생성된다.


  ```bash
  $ ./metricbeat -e
  ```

  - 여기까지 실행하면 Elasticsearch에 `metricbeat-<버전>-<날짜>-000001` 형식으로 인덱스가 생성 된 것을 확인 가능하다.



- docker container로 실행하기

  - 이미지를 받는다.

  > https://www.docker.elastic.co/r/beats

  - docker container 실행하기

  ```yaml
  version: '3.2'
  
  services:
    metricbeat:
      image: <위에서 받은 image>
      user: root
      environment:
        ELASTICSEARCH_HOSTS: http://<elasticsearch_host:port>
        KIBANA_HOST: http://<kibana_host:port>
      networks:
        - <elasticsearch_network>
  
  volumes:
    data01:
      driver: local
  ```



- `yml` 파일 볼륨 설정하기

  - `metricbeat.yml` 파일과 같이 config 파일을 volume을 잡으면 metricbeat이 실행될 때 `config file must be owned by the user identifier (uid=0) or root`와 같은 error가 뜬다.
  - 이는 metricbeat을 실행할 때 `-strict.perms=false` 플래그를 주면 해결된다.
    - 혹은 볼륨 잡을 파일의 권한을 변경해도 된다.

  ```yaml
  version: '3.2'
  
  services:
    metricbeat:
      image: docker.elastic.co/beats/metricbeat:7.16.0
      user: root
      environment:
        ELASTICSEARCH_HOSTS: http://127.0.0.1:9202
        KIBANA_HOST: http://127.0.0.1:5603
        TZ: Asia/Seoul
      # 아래와 같이 config 파일을 volume 잡으면 권한 관련 error가 발생한다.
      volumes:
        - /home/theo/workspace/practice/elasticsearch/metricbeat/metricbeat.yml:/usr/share/metricbeat/metricbeat.yml:ro
      # 따라서 metricbeat을 실행할 때 -strict.perms=false 옵션을 줘야 한다.
      entrypoint: ./metricbeat -e -strict.perms=false
  
  networks:
    default:
      external:
        name: elasticsearch_elastic
  ```



- 모듈 비활성화하기

  ```bash
  $ ./metricbeat modules disable <모듈명>
  ```



- docker-compose 로 띄울 때, 새로운 모듈 추가하기

  - elasticsearch에서 배포한 [metricbeat 이미지](https://www.docker.elastic.co/r/beats)는 기본적으로 system module만 모니터링하도록 설정되어 있다.
  - metricbeat은 `modules.d`디렉터리에 `.disabled`가 붙지 않은 파일의 존재 여부로 module이 활성화 되었는지를 판단한다(추측).
  - 따라서 `modules.d` 디렉터리에 `.disabled`가 붙지 않은 yml 파일을 추가해준다.
  - `elasticsearch.yml` 파일 생성

  ```yaml
  - module: elasticsearch
    metricsets:
      - node
      - node_stats
      - index
    period: 10s
    hosts: '${ELASTICSEARCH_HOSTS}'
  ```

  - volume 생성
    - volume만 생성하면 `./metricbeat module enable elasticasearch` 명령을 입력하지 않아도 enable 상태가 된다.
    - 만일 volume을 잡고도 위 명령어를 입력하면 이미 enable 상태라는 메시지가 뜬다.

  ```yaml
  version: '3.2'
  
  services:
    metricbeat:
      image: docker.elastic.co/beats/metricbeat:7.16.0
      user: root
      environment:
        ELASTICSEARCH_HOSTS: http://127.0.0.1:9202
        KIBANA_HOST: http://127.0.0.1:5603
        TZ: Asia/Seoul
      volumes:
        - ./metricbeat.yml:/usr/share/metricbeat/metricbeat.yml
        # 볼륨을 추가해준다.
        - ./elasticsearch.yml:/usr/share/metricbeat/modules.d/elasticsearch.yml
      entrypoint: bash -c "./metricbeat -e -strict.perms=false"
  
  networks:
    default:
      external:
        name: elasticsearch_elastic
  ```



- metricbeat은 cluster 내의 모든 node에 연결하는 것이 좋다.

  - master node에만 연결한다고 동작하지 않는 것은 아니지만, 모든 node에 연결하는 것이 권장된다.
  - `elasticsearch.yml` 설정에 아래와 같이 모든 node 정보를 추가한다.

  ```yaml
  - module: elasticsearch
    metricsets:
      - index
    period: 10s
    hosts: ["<node1의 host:port>", "<node2의 host:port>", "<node3의 host:port>"]
  ```



- elasticsearch monitoring하기

  - `elasticsearch-xpack` 모듈을 사용한다.

  ```bash
  $ ./metricbeat modules enable elasticsearch-xpack
  ```

  - `elasticsearch-xpack.yml` 파일을 설정한다.
    - `elasticsearch.yml` 파일과 거의 동일하다.
    - 단, `metricsets`이 빠지고 `xpack.enabled`이 추가된다.

  ```yaml
  - module: elasticsearch
    xpack.enabled: true
    period: 10s
    hosts: ["<node1의 host:port>", "<node2의 host:port>", "<node3의 host:port>"]
  ```

  - 다른 module이 `metricbeat-<es_version>-<날짜>` 형식의 인덱스에 저장되는 것과는 달리 elasticsearch-xpack 모듈은 `.monitoring-es-<version>-mb-<날짜>` 형식의 index에 저장된다.
    - elasticsearch에서 기본적으로 제공하는 monitoring 기능을 사용하면 `.monitoring-es-<version>-<날짜>` 형식의 인덱스에 저장이 되는데 여기에 `mb`(metricbeat)만 추가된 것이다.
    - elasticsearch에서 기본적으로 제공하는 monitoring 기능은 곧 deprecated 될 예정이다.

