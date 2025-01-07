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





# Service를 사용하여 Application들 연결하기

- Kubernetes는 모든Pod들에게 cluster-private IP를 제공한다.
  - 따라서 명시적으로 Pod들 사이에 link를 설정하거나, conatiner port와 host port를 mapping할 필요가 없다.
  - 이는 Pod 내의 container들이 localhost를 통해 각각의 port로 접근할 수 있으며, cluster 내의 모든 Pod들이 NAT 없이도 연결될 수 있다는 것을 의미한다.



- Nginx Pod를 생성한다.

  - Manifest 작성

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: my-nginx
  spec:
    selector:
      matchLabels:
        run: my-nginx
    replicas: 2
    template:
      metadata:
        labels:
          run: my-nginx
      spec:
        containers:
        - name: my-nginx
          image: nginx
          ports:
          - containerPort: 80
  ```

  - 위에서 작성한 file로 Pod를 생성한다.

  ```bash
  # 생성
  $ kubectl apply -f ./run-my-nginx.yaml
  
  # 확인
  $ kubectl get pods -l run=my-nginx -o wide
  # output
  NAME                        READY     STATUS    RESTARTS   AGE       IP            NODE
  my-nginx-3800858182-jr4a2   1/1       Running   0          13s       10.244.3.4    kubernetes-minion-905m
  my-nginx-3800858182-kna2y   1/1       Running   0          13s       10.244.2.5    kubernetes-minion-ljyd
  ```

  - Pod의 IP 확인
    - Container들이 80 port를 사용하지 않으며, traffic을 Pod에 routing하기 위한 특별한 NAT rule도 없다는 것을 알아야한다.
    - 이는 같은 `containerPort`를 사용하는 여러 개의 Nginx Pod를 같은 node에서 실행할 수 있다는 것을 의미한다.
    - 또한 cluster 내의 다른 Pod 혹은 node에서도 Pod에 할당된 IP를 통해 접근이 가능하다.

  ```bash
  $ kubectl get pods -l run=my-nginx -o custom-columns=POD_IP:.status.podIPs
  
  # output
  POD_IP
  [map[ip:10.244.3.4]]
  [map[ip:10.244.2.5]]
  ```



- Service 생성하기

  - Service를 생성해야하는 이유
    - 이론적으로는 Pod가 할당받은 IP로 바로 접근할 수 있다.
    - 그러나, Pod가 내려갈 경우 ReplicaSet은 새로운 IP를 할당 받은 새로운 Pod를 생성할 것이다.
    - 따라서 기존의 IP로는 더 이상 Pod에 접근할 수 없게 된다.
  - Service가 생성되면, 각 Service는 clusterIP라 불리는 고유한 IP address를 부여 받는다.
    - 이 address는 Service의 lifespan과 함께하며, Service가 살아있는 동안에는 변하지 않는다.
    - Service로 들어오는 모든 요청은 Service로 묶여있는 Pod들로 load-balancing된다.
  - Service 생성하기
    - Cluster 내의 어떤 node에서든 `cluster_IP:port`로 Nginx Service에 요청을 보낼 수 있다.
    - Service IP는 완전한 가상 IP이다.

  ```bash
  $ kubectl expose deployment/my-nginx
  ```

  - 위 명령어는 아래와 같은 manifest 파일을 apply하는 것과 동일하다.
    - 아래 spec은 `run: my-nginx`라는 label을 가지고 있는 모든 Pod에게 port 80을 대상으로하는 Service를 생성한다.
    - `targetPort`는 container가 traffic을 받아들이는 port이다.
    - `port`는 다른 Pod들이 Service에 접근할 때 사용하는 port이다.

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-nginx
    labels:
      run: my-nginx
  spec:
    ports:
    - port: 80
      protocol: TCP
    selector:
      run: my-nginx
  ```

  - Service 확인하기
    - Service에 속한 Pod들은 EndpointSlice에 의해 expose된다.
    - Service의 selector는 지속적으로 평가되며, 그 결과는 label에 의해 Service와 연결되어 있는 EndpointSlice에게 POST된다.
    - Pod가 죽을 경우, EndPointSlice에서 자동으로 제거되며, 새로 생성된 Pod가 자동으로 추가된다.

  ```bash
  $ kubectl get svc my-nginx
  
  # output
  NAME       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
  my-nginx   ClusterIP   10.0.162.149   <none>        80/TCP    21s
  ```

  - EndpointSlice 확인

  ```bash
  $ kubectl describe svc my-nginx
  # output
  Name:                my-nginx
  Namespace:           default
  Labels:              run=my-nginx
  Annotations:         <none>
  Selector:            run=my-nginx
  Type:                ClusterIP
  IP Family Policy:    SingleStack
  IP Families:         IPv4
  IP:                  10.0.162.149
  IPs:                 10.0.162.149
  Port:                <unset> 80/TCP
  TargetPort:          80/TCP
  Endpoints:           10.244.2.5:80,10.244.3.4:80
  Session Affinity:    None
  Events:              <none>
  
  $ kubectl get endpointslices -l kubernetes.io/service-name=my-nginx
  # output
  NAME             ADDRESSTYPE   PORTS   ENDPOINTS               AGE
  my-nginx-7vzhx   IPv4          80      10.244.2.5,10.244.3.4   21
  ```



- 환경 변수

  - Kubelet은 Pod에 Service와 관련된 environment variable들을 추가한다.
    - 아래와 같이 확인이 가능하다.

  ```bash
  $ kubectl exec my-nginx-3800858182-jr4a2 -- printenv | grep SERVICE
  
  # output
  KUBERNETES_SERVICE_HOST=10.0.0.1
  KUBERNETES_SERVICE_PORT=443
  KUBERNETES_SERVICE_PORT_HTTPS=443
  ```

  - 위 결과에는 Service에 대한 설명이 없다.
    - 이는 Service보다 replica가 먼저 생성되었기 때문이다.
    - Pod에 Service와 관련된 환경 변수를 추가하기 위해, Pod들을 전부 죽이고 새로 생성한다.

  ```bash
  # replica의 개수를 0으로 설정해 Pod를 전부 없앤다.
  $ kubectl scale deployment my-nginx --replicas=0
  
  # 이후 다시 2개로 늘려 새로 생성한다.
  $ kubectl scale deployment my-nginx --replicas=2
  ```

  - 다시 환경 변수를 확인한다.
    - Service와 관련된 환경 변수가 추가된 것을 확인할 수 있다.

  ```bash
  $ kubectl exec my-nginx-3800858182-e9ihh -- printenv | grep SERVICE
  
  # output
  KUBERNETES_SERVICE_PORT=443
  MY_NGINX_SERVICE_HOST=10.0.162.149
  KUBERNETES_SERVICE_HOST=10.0.0.1
  MY_NGINX_SERVICE_PORT=80
  KUBERNETES_SERVICE_PORT_HTTPS=443
  ```



- Service 보안 설정

  - 현재까지는 cluster 내부에서만 Nginx에 접근으나, Service를 internet에 노출시키기 전에, communication channel이 안전한지 확인해야한다.
    - HTTPS를 위해 self signed certificates를 생성한다.
    - Nginx가 certificates를 사용하도록 설정한다.
    - Pod에서 certificate에 접근할 수 있도록 Secret을 생성한다.
  - Secret 생성

  ```bash
  $ make keys KEY=/tmp/nginx.key CERT=/tmp/nginx.crt
  $ kubectl create secret tls nginxsecret --key /tmp/nginx.key --cert /tmp/nginx.crt
  ```

  - Secret 확인

  ```bash
  $ kubectl get secrets
  
  # output
  NAME                  TYPE                                  DATA      AGE
  nginxsecret           kubernetes.io/tls                     2         1m
  ```

  - ConfigMap 생성을 위한 default.conf파일 작성

  ```ini
  server {
          listen 80 default_server;
          listen [::]:80 default_server ipv6only=on;
  
          listen 443 ssl;
  
          root /usr/share/nginx/html;
          index index.html;
  
          server_name localhost;
          ssl_certificate /etc/nginx/ssl/tls.crt;
          ssl_certificate_key /etc/nginx/ssl/tls.key;
  
          location / {
                  try_files $uri $uri/ =404;
          }
  }
  ```

  - ConfigMap 생성

  ```bash
  $ kubectl create configmap nginxconfigmap --from-file=default.conf
  ```

  - ConfigMap 확인

  ```bash
  $ kubectl get configmaps
  
  # output
  NAME             DATA   AGE
  nginxconfigmap   1      114s
  ```

  - ConfigMap 상세 보기

  ```bash
  $ kubectl describe configmap  nginxconfigmap
  
  # output
  Name:         nginxconfigmap
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>
  
  Data
  ====
  default.conf:
  ----
  server {
          listen 80 default_server;
          listen [::]:80 default_server ipv6only=on;
  
          listen 443 ssl;
  
          root /usr/share/nginx/html;
          index index.html;
  
          server_name localhost;
          ssl_certificate /etc/nginx/ssl/tls.crt;
          ssl_certificate_key /etc/nginx/ssl/tls.key;
  
          location / {
                  try_files $uri $uri/ =404;
          }
  }
  
  BinaryData
  ====
  
  Events:  <none>
  ```

  - Nginx replica가 certificate을 사용하여 HTTPS server를 실행하도록 변경하기

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-nginx
    labels:
      run: my-nginx
  spec:
    type: NodePort
    ports:
    - port: 8080
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      protocol: TCP
      name: https
    selector:
      run: my-nginx
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: my-nginx
  spec:
    selector:
      matchLabels:
        run: my-nginx
    replicas: 1
    template:
      metadata:
        labels:
          run: my-nginx
      spec:
        volumes:
        - name: secret-volume
          secret:
            secretName: nginxsecret
        - name: configmap-volume
          configMap:
            name: nginxconfigmap
        containers:
        - name: nginxhttps
          image: bprashanth/nginxhttps:1.0
          ports:
          - containerPort: 443
          - containerPort: 80
          volumeMounts:
          - mountPath: /etc/nginx/ssl
            name: secret-volume
          - mountPath: /etc/nginx/conf.d
            name: configmap-volume
  ```

  - 삭제 후 다시 생성하기

  ```bash
  # 삭제
  $ kubectl delete deployments,svc my-nginx
  
  # 재생성
  $ kubectl create -f ./nginx-secure-app.yaml
  ```

  - 확인하기

  ```bash
  $ kubectl get pods -l run=my-nginx -o custom-columns=POD_IP:.status.podIPs
  # output
  POD_IP
  [map[ip:10.244.3.5]]
  
  $ curl -k https://10.244.3.5
  # output
  <h1>Welcome to nginx!</h1>
  ```

