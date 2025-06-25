# Airbyte

- Airbyte

  - Airflow 개발자들이 만든 ETL tool이다.
    - Source system에서 destination system으로 data를 옮기기 위해 사용한다.
    - 이를 위해서 주기적으로 source로 부터 data를 읽어서 추출된 data들을 destination으로 옮기는 sync run을 수행한다.
    - Airbyte의 주요 이점중 하나는 수백개의 서로 다른 source들로부터 data를 추출하여 다시 많은 수의 destination으로 load할 수 있다는 점이다.
    - sync run이 실행되는 주기를 직접 지정할 수도 있지만, Airflow, Dagster, Prefect 등의 orchestrator에게 넘길 수도 있다.
    - Orchecstrator에게 sync run을 실행하는 scheduling을 맡길 경우 sync run이 실행되기 전에 수행해야 하는 작업이나, sync run 이후에 수행해야 하는 작업을 더 잘 조정할 수 있게 된다.
  - 변환 기능이 풍부하진 않아서, 데이터 변환을 위해서 일반적으로 dbt(Data Build Tool)라 불리는 데이터 변환 도구를 함께 사용한다.

  - CDC 지원
    - 현재까지는 MySQL, MongoDB, PostgreSQL, MSSQL만 지원한다.



- Airflow
  - 일련의 task들을 schedule에 맞춰 실행하도록 하는 tool이다.
    - 어떤 task들이 다른 task들이 완료된 이후에 실행되어야 할 때 특히 유용하게 사용할 수 있다.
  - Airflow가 ETL(혹은 ELT) tool인가?
    - Airflow는 내장 operator과 hook들, 그리고 community에서 관리되는 operator들과 hook들을 제공한다.
    - 이를 이용하여 많은 종류의 task들을 실행하고 trigger할 수 있다.
    - 그러나 Airflow는 ochestration에 목적이 있는 tool이지 ETL이 주요 목적인 tool은 아니다.
    - 물론 Airflow는 여러 task를 scheduling 할 수 있고, Airflow가 scheduling하는 task들이 ETL task일 수는 있다.
    - 그러나 Airflow 자체는 ETL만을 염두에 두고 만들어지지는 않았다.



- Airbyte와 Airflow의 차이
  - 목적
    - Airbyte는 데이터를 한 시스템에서 다른 시스템으로 옮기는 도구이다.
    - Airflow는 일련의 작업을 특정한 순서대로 주기적으로 실행하고 스케줄링하는 orchestrator 도구이다.
  - Airbyte와 Airflow를 함께 사용하는 이유
    - Airbyte 역시 자체적인 스케줄러를 가지고 있다.
    - 그럼에도 Airflow와 같은 orchestrator를 사용하는 이유는 Airflow가 Airbyte가 데이터 이동을 실행하기 전후로 실행되야 하는 작업들도 함께 조정해주기 때문이다.



# Airbyte 실행하기

- Docker로 Airbyte 실행하기

  - Github preository에서 Airbyte를 clone 받는다.

  ```bash
  $ git clone --depth=1 https://github.com/airbytehq/airbyte.git
  ```

  - `run-ab-platform.sh`를 실행한다.
    - `run-ab-platform.sh`가 실행되면, Docker container를 실행하는 데 필요한 docker-compose.yml 파일들이 다운로드 되고, 해당 파일을 사용하여 container들을 실행한다.

  ```bash
  $ cd airbyte
  $ ./run-ab-platform.sh
  ```

  - Web UI로 접속한다.
    - 기본 계정은 airbyte / password로 설정되어 있다.





# Destination

## Iceberg

- Iceberg 실행하기

  - 아래와 같이 docker-compose.yml 파일을 작성한다.

  ```yaml
  version: "3"
  
  x-minio-common: &minio-common
    image: minio/minio:latest
    command: server --console-address ":9001" http://minio{1...2}/data{1...2}
    environment:
      MINIO_ROOT_USER: "foo"
      MINIO_ROOT_PASSWORD: "foo_auth"
      MINIO_STORAGE_CLASS_STANDARD: "EC:1"
      MINIO_DOMAIN: "minio"
    restart: always
  
  services:
    spark-iceberg:
      image: tabulario/spark-iceberg
      container_name: spark-iceberg
      depends_on:
        - rest
      environment:
        - AWS_ACCESS_KEY_ID=foo
        - AWS_SECRET_ACCESS_KEY=foo_auth
        - AWS_REGION=us-east-1
      ports:
        - 8888:8888
        - 8080:8080
        - 10000:10000
        - 10001:10001
      networks:
        iceberg_net:
    
    rest:
      image: tabulario/iceberg-rest
      container_name: iceberg-rest
      ports:
        - 8181:8181
      environment:
        - AWS_ACCESS_KEY_ID=foo
        - AWS_SECRET_ACCESS_KEY=foo_auth
        - AWS_REGION=us-east-1
        - CATALOG_WAREHOUSE=s3a://elt/test
        - CATALOG_IO__IMPL=org.apache.iceberg.aws.s3.S3FileIO
        # 아직 원인은 모르겠지만 CATALOG_S3_ENDPOINT 값을 container network의 내부 주소로 설정하면 동작하지 않는다.
        # 즉 http://minio1:9000과 같이 설정하면 동작하지 않는다.
        - CATALOG_S3_ENDPOINT=http://<host_ip>:9020
      networks:
        iceberg_net:
  
    minio1:
      <<: *minio-common
      container_name: minio1
      networks:
        iceberg_net:
          aliases:
            - elt.minio
      ports:
        - 9020:9000
        - 9021:9001
  
    minio2:
      <<: *minio-common
      container_name: minio2
      networks:
        iceberg_net:
      ports:
        - 9030:9000
        - 9031:9001
  
  networks:
    iceberg_net:
  ```

  - Docker container를 실행한다.

  ```bash
  $ docker compose up
  ```

  - MinIO가 실행되면 MinIO UI로 접속하여 아래와 같은 작업을 실행해야한다.
    - `Settings`-`Region`에서 `Server Location`을 설정한다.
    - 새로운 bucket을 생성하고, bucket의 `Access Policy`를 `public`으로 변경한다.
    - Bucket 내부에 warehouse로 사용할 path를 생성한다.
    - 위 예시의 경우 bucket 이름은 `elt`, warehouse는 `test`로 설정했다.



- Airbyte에서 `Apache Iceberg` destination을 생성한다.

  - `Iceberg Catalog config`에서는 `HadoopCatalog`를 선택한다.

  - `Storage config`는 `S3`를 선택하고 아래와 같이 설정해준다.
    - `S3 Key ID`에는 MinIO container를 실행할 때 설정했던 `MINIO_ROOT_USER`값을, `S3 Access Key`에는 `MINIO_ROOT_PASSWORD` 값을 넣으면 된다.
    - `S3 Warehouse Uri for Iceberg`에는 `s3a://<bucket>/<warehouse_path>` 형식으로 입력하면 되며, 주의할 점은 `s3`가 아닌 `s3a`를 사용해야 한다는 점과, `<warehouse_path>` 없이 bucket만 입력할 경우 error가 발생한다는 점이다.
    - `S3 Bucket Region`에는 MinIO를 실행한 후 설정했던 `Server Location`과 동일한 값을 넣어야 한다.
    - `Endpoint`를 입력할 때는 반드시 `http`, `https` 등의 프로토콜도 함께 입력해야 한다.

  ![image-20240404103108042](Airbyte.assets/image-20240404103108042.png)



