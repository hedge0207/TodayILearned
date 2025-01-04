# PV 예시 - PV를 사용하여 WordPress와 MySQL 배포하기

- PVC와 PV 생성하기

  - WordPress와 MySQL 모두 PV와 PVC를 사용한다.

    - PV는 **관리자**에 의해 provisioning되거나 Kubernetes에 의해 동적으로 provisioning 된 cluster 내부의 storage로, StorageClass를 사용한다.

    - PVC는 **사용자**가 PV에 보내는 요청이다.

  - `kustomization.yaml` 파일 작성하기

  ```yaml
  secretGenerator:
  - name: mysql-pass
    literals:
    - password=<password>
  ```

  - MySQL을 위한 resource config 작성하기
    - Service, PVC, Deployment를 생성한다.
    - MySQL container는 PV를 ` /var/lib/mysql`에 mount한다.
    - `MYSQL_ROOT_PASSWORD` 환경 변수는 Secret으로부터 DB password를 설정한다.

  ```yaml
  # mysql-deployment.yml
  apiVersion: v1
  kind: Service
  metadata:
    name: wordpress-mysql
    labels:
      app: wordpress
  spec:
    ports:
      - port: 3306
    selector:
      app: wordpress
      tier: mysql
    clusterIP: None
  ---
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: mysql-pv-claim
    labels:
      app: wordpress
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 20Gi
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: wordpress-mysql
    labels:
      app: wordpress
  spec:
    selector:
      matchLabels:
        app: wordpress
        tier: mysql
    strategy:
      type: Recreate
    template:
      metadata:
        labels:
          app: wordpress
          tier: mysql
      spec:
        containers:
        - image: mysql:8.0
          name: mysql
          env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-pass
                key: password
          - name: MYSQL_DATABASE
            value: wordpress
          - name: MYSQL_USER
            value: wordpress
          - name: MYSQL_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-pass
                key: password
          ports:
          - containerPort: 3306
            name: mysql
          volumeMounts:
          - name: mysql-persistent-storage
            mountPath: /var/lib/mysql
        volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim
  ```

  - WordPress를 위한 resource config 작성하기
    - WordPress container는 PV를 `/var/www/html`에 mount한다.
    - `WORDPRESS_DB_HOST` 환경 변수는 위에서 생성한 MySQL Service의 이름을 설정하며, Service를 통해 MySQL에 접근한다.
    - `WORDPRESS_DB_PASSWORD` 환경 변수는 Secret으로부터 DB password를 설정한다.

  ```yaml
  # wordpress-deployment.yml
  apiVersion: v1
  kind: Service
  metadata:
    name: wordpress
    labels:
      app: wordpress
  spec:
    ports:
      - port: 80
    selector:
      app: wordpress
      tier: frontend
    type: LoadBalancer
  ---
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: wp-pv-claim
    labels:
      app: wordpress
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 20Gi
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: wordpress
    labels:
      app: wordpress
  spec:
    selector:
      matchLabels:
        app: wordpress
        tier: frontend
    strategy:
      type: Recreate
    template:
      metadata:
        labels:
          app: wordpress
          tier: frontend
      spec:
        containers:
        - image: wordpress:6.2.1-apache
          name: wordpress
          env:
          - name: WORDPRESS_DB_HOST
            value: wordpress-mysql
          - name: WORDPRESS_DB_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-pass
                key: password
          - name: WORDPRESS_DB_USER
            value: wordpress
          ports:
          - containerPort: 80
            name: wordpress
            
          volumeMounts:
          - name: wordpress-persistent-storage
            mountPath: /var/www/html
        volumes:
        - name: wordpress-persistent-storage
          persistentVolumeClaim:
            claimName: wp-pv-claim
  ```

  - `kustomization.yaml` 파일에 위의 두 yaml 파일을 추가한다.

  ```yaml
  secretGenerator:
  - name: mysql-pass
    literals:
    - password=<password>
  resources:
    - mysql-deployment.yaml
    - wordpress-deployment.yaml
  ```



- 적용하고 확인하기

  - `kubectl apply` 실행
    - `kustomization.yaml`파일은 WordPress와 MySQL을 배포하기 위한 모든 정보를 담고 있다.

  ```bash
  $ kubectl apply -k ./
  ```

  - 모든 object가 생성되었는지 확인하기

  ```bash
  # Secret 생성 여부 확인
  $ kubectl get secrets
  
  # PV provisioning 여부 확인
  $ kubectl get pvc
  
  # Pod 생성 여부 확인
  $ kubectl get pods
  
  # Service 생성 여부 확인
  $ kubectl get services wordpress
  ```

  - WordPress 접속하기
    - 아래 명령어를 실행하면 출력되는 URL로 접속한다.

  ```bash
  $ minikube service wordpress --url
  ```





# 
