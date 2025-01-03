# Configuration

- 아래 내용을 테스트하기 위해서 control plane 역할을 하지 않는 2개의 노드를 cluster에 포함시키는 것이 권장된다.

  - minikube로 cluster 생성하기

  ```bash
  $ minikube start --nodes 3
  ```

  - 확인하기

  ```bash
  $ kubectl get nodes
  ```





## ConfigMap을 통해 설정 변경하기

- Volume으로 mount된 ConfigMap으로 설정 변경하기

  - 현재 ConfigMap 확인

  ```bash
  $ kubectl get configmaps
  ```

  - `kubectl create configmap`에 `--from-literal` 옵션을 주어 literal value를 사용하여 ConfigMap을 생성할 수 있다.

  ```bash
  $ kubectl create configmap <configmap_name> --from-literal=<key>=<value>
  
  # e.g.
  $ kubectl create configmap sport --from-literal=sport=football
  ```

  - 위에서 생성한 ConfigMap 확인

  ```bash
  $ kubectl get configmaps sport -o yaml
  ```

  - 출력되는 결과는 아래와 같다.
    - 위에서 literal value로 입력한 `sport=football`이 `data`에 추가된 것을 확인할 수 있다.

  ```yaml
  apiVersion: v1
  data:
    sport: football
  kind: ConfigMap
  metadata:
    creationTimestamp: "2024-11-27T01:20:05Z"
    name: sport
    namespace: default
    resourceVersion: "5475"
    uid: 5gh6484c-1d27-5t8d-asb1-a53f7wra6c2
  ```

  - Deployment를 생성한다.
    - `kubectl apply`의 경우 `kubectl create`와 유사하게 자원을 생성하는 명령어이다.
    - 다만 `kubectl apply`의 경우 생성하려는 자원이 없으면 자원을 생성하고, 자원이 이미 있으면 자원의 설정을 수정한다.
    - `-f` option 뒤에는 설정을 적용할 file 경로를 입력한다.

  ```bash
  $ kubectl apply -f https://k8s.io/examples/deployments/deployment-with-configmap-as-volume.yaml
  ```

  - 위에서 사용한 `deployments/deployment-with-configmap-as-volume.yaml`의 내용은 아래와 같다.
    - `spec.template.spec.containers.command`를 보면 `/etc/config/sport` 파일의 내용을 주기적으로 출력하는 application이라는 것을 알 수 있다.

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: configmap-volume
    labels:
      app.kubernetes.io/name: configmap-volume
  spec:
    replicas: 3
    selector:
      matchLabels:
        app.kubernetes.io/name: configmap-volume
    template:
      metadata:
        labels:
          app.kubernetes.io/name: configmap-volume
      spec:
        containers:
          - name: alpine
            image: alpine:3
            command:
              - /bin/sh
              - -c
              - while true; do echo "$(date) My preferred sport is $(cat /etc/config/sport)";
                sleep 10; done;
            ports:
              - containerPort: 80
            volumeMounts:
              - name: config-volume
                mountPath: /etc/config
        volumes:
          - name: config-volume
            configMap:
              name: sport
  ```

  - 위에서 생성한 Deployment의 Pod들을 확인한다.
    - Selector를 사용하여 Pod를 matching시킨다.
    - 위 설정 파일에서 `metadata.labels`의 값을 `--selector` option에 준다.

  ```bash
  $ kubectl get pods --selector=app.kubernetes.io/name=configmap-volume
  
  # output
  NAME                                READY   STATUS    RESTARTS   AGE
  configmap-volume-6b976dfdcf-qxvbm   1/1     Running   0          72s
  configmap-volume-6b976dfdcf-skpvm   1/1     Running   0          72s
  configmap-volume-6b976dfdcf-tbc6r   1/1     Running   0          72s
  ```

  - Kubelet은 위 Pod들이 실행 중인 각 Node들에 ConfigMap에 대한 데이터를 가져와 local volume의 file로 변환한다.
    - 이후 kubelete은 위 설정 파일의 `spec.template.spec.container.volumeMounts`에 정의된대로 volume을 container에 mount한다.
    - 해당 container는 mount된 file에서 정보를 로드하여 `spec.template.spec.containers.command`를 실행시킬 때 사용한다.
    - Deployment에 속한 Pod의 log를 보면 출력 결과를 확인할 수 있다.

  ```bash
  $ kubectl logs deployments/configmap-volume
  
  # output
  Found 3 pods, using pod/configmap-volume-6b3qd48f2-qd1n8
  Wed Nov 27 01:28:42 UTC 2024 My preferred sport is football
  Wed Nov 27 01:28:52 UTC 2024 My preferred sport is football
  Wed Nov 27 01:29:02 UTC 2024 My preferred sport is football
  Wed Nov 27 01:29:12 UTC 2024 My preferred sport is football
  Wed Nov 27 01:29:22 UTC 2024 My preferred sport is football
  ```

  - Mount된 파일 확인하기
    - Pod 중 하나의 `/etc/config/sport`를 확인한다.

  ```bash
  $ kubectl exec -it configmap-volume-7b4bb95f8-n674r -- cat /etc/config/sport
  
  # output
  football
  ```

  - ConfigMap 수정하기
    - 아래 명령어를 입력하면 ConfigMap을 수정할 수 있는 editor가 실행된다.
    - Editor에서 `data.sport`의 값을 football에서 baseball로 변경한 뒤 창을 닫는다.
    - 성공적으로 변경 되면 `configmap/sport edited`와 같은 message가 출력된다.

  ```bash
  $ kubectl edit configmap sport
  ```

  - 다시 log를 확인한다.
    - 이전과는 달라진 것을 확인할 수 있다.

  ```bash
  $ kubectl logs deployments/configmap-volume --follow
  
  # output
  Wed Nov 27 01:52:43 UTC 2024 My preferred sport is football
  Wed Nov 27 01:52:53 UTC 2024 My preferred sport is football
  Wed Nov 27 01:53:03 UTC 2024 My preferred sport is baseball
  Wed Nov 27 01:53:13 UTC 2024 My preferred sport is baseball
  Wed Nov 27 01:53:23 UTC 2024 My preferred sport is baseball
  ```

  - 설정의 변경 사항이 항상 반영되는 것은 아닐 수 있다.
    - `configMap` volume이나 `projected` volume을 통해 실행 중인 Pod에 mapping된 ConfigMap이 있다면, ConfigMap의 변경 사항이 실행 중인 Pod에 반영될 것이다.
    - 그러나 app;ication은 변경 사항을 polling하거나 file의 update를 감시하도록 설정된 경우에만 변경 사항을 알 수 있다.
    - 만약 application이 최초 실행될 때만 설정을 load한다면, 설정의 변경 사항이 반영되지 않을 수 있다.

  - 변경된 설정이 반영되는 기간
    - ConfigMap이 수정되고, 변경 사항이 Pod에 반영되기 까지는 kubelet sync period만큼의 시간이 걸린다.
    - Kubelet은 mount된 ConfigMap에 변경 사항이 있는지 매 periodic sync마다 체크한다.
    - 그러나 kubelet은 현재 ConfigMap의 값을 가져오기 위해 자신의 local TTL-based cahce를 사용한다.
    - 그로 인해 ConfigMap이 수정되고, 변경 사항이 Pod에 반영되기 까지는 kubelet sync period(default: 1m) + ConfigMaps cache의 TTL(default: 1m)까지 걸릴 수 있다.



- ConfigMap을 통해 Pod의 환경 변수 수정하기

  - 위에서와 마찬가지로 literal value를 사용하여 ConfigMap을 생성한다.

  ```bash
  $ kubectl create configmap fruits --from-literal=fruits=apples
  ```

  - Deployment를 생성한다.

  ```bash
  $ kubectl apply -f https://k8s.io/examples/deployments/deployment-with-configmap-as-envvar.yaml
  ```

  - 위에서 Deployment 생성에 사용한 `deployments/deployment-with-configmap-as-envvar.yaml` 파일은 아래와 같다.
    - `spec.template.spec.containers.command`를 보면 환경 변수 `$FRUITS`를 주기적으로 출력하는 application이라는 것을 알 수 있다.
    - 이전과는 달리 volume을 생성하지는 않는다.

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: configmap-env-var
    labels:
      app.kubernetes.io/name: configmap-env-var
  spec:
    replicas: 3
    selector:
      matchLabels:
        app.kubernetes.io/name: configmap-env-var
    template:
      metadata:
        labels:
          app.kubernetes.io/name: configmap-env-var
      spec:
        containers:
          - name: alpine
            image: alpine:3
            env:
              - name: FRUITS
                valueFrom:
                  configMapKeyRef:
                    key: fruits
                    name: fruits
            command:
              - /bin/sh
              - -c
              - while true; do echo "$(date) The basket is full of $FRUITS";
                  sleep 10; done;
            ports:
              - containerPort: 80
  ```

  - 위에서 생성한 Deployment에 대한 Pod를 확인한다.

  ```bash
  $ kubectl get pods --selector=app.kubernetes.io/name=configmap-env-var
  ```

  - 출력되는 내용을 확인한다.

  ```bash
  $ kubectl logs deployment/configmap-env-var
  
  # output
  Found 3 pods, using pod/configmap-env-var-7c994f7769-l74nq
  Wed Nov 27 02:17:24 UTC 2024 The basket is full of apples
  Wed Nov 27 02:17:34 UTC 2024 The basket is full of apples
  Wed Nov 27 02:17:44 UTC 2024 The basket is full of apples
  ```

  - ConfigMap을 수정한다.
    - editor에서 `data.fruits`의 값을 watermelons으로 수정한다.

  ```bash
  $ kubectl edit configmap fruits
  ```

  - 변경 사항이 적용되었는지 확인한다.
    - 변경 사항이 적용되지 않은 것을 확인할 수 있다.

  ```bash
  $ kubectl logs deployments/configmap-env-var --follow
  ```

  - 변경 사항이 적용되지 않는 이유
    - ConfigMap 내부의 key에 해당하는 value가 변경되었지만, Pod 내부의 환경 변수는 여전히 이전 값을 보여준다.
    - 이는 Pod 내부에서 실행 중인 process의 환경 변수가 source data의 변경으로는 변경되지 않기 때문이다.
    - 만약 강제로 변경하고자 한다면 Kubernetes가 기존의 Pod를 교체하도록 해야 한다.
    - 새로 교체된 Pod에는 변경 사항이 반영된다.
  - Pod 교체하기
    - Rollout은 Kubernetes가 Deployment에 대한 새로운 ReplicaSet을 만들도록 한다.
    - 이는 기존의 Pod들이 종료되고, 새로운 Pod들이 생성된다는 것을 의미한다.

  ```bash
  $ kubectl rollout restart deployment configmap-env-var
  
  # rollout이 완료되면 아래 명령어를 수행한다.
  $ kubectl rollout status deployment configmap-env-var --watch=true
  ```

  - Deployment와 Pod를 확인한다.
    - Deployment의 상태와 Pod들이 잘 교체되었는지 확인한다.

  ```bash
  $ kubectl get deployment configmap-env-var
  $ kubectl get pods --selector=app.kubernetes.io/name=configmap-env-var
  ```

  - 다시 log를 확인한다.
    - 잘 적용된 것을 확인할 수 있다.

  ```bash
  $ kubectl logs deployment/configmap-env-var
  
  # output
  Found 3 pods, using pod/configmap-env-var-694f488fd-rhkh6
  Wed Nov 27 02:34:57 UTC 2024 The basket is full of watermelons
  Wed Nov 27 02:35:07 UTC 2024 The basket is full of watermelons
  Wed Nov 27 02:35:17 UTC 2024 The basket is full of watermelons
  Wed Nov 27 02:35:27 UTC 2024 The basket is full of watermelons
  ```



- Multi-container Pod에서 ConfigMap을 통해 설정 변경하기

  - Multi-container Pod
    - 2개 이상의 서로 다른 container를 포함하고 있는 Pod를 의미한다.
    - 일반적으로 하나의 Pod 안에서는 하나의 process가 하나의 container를 구동한다.
    - 그러나 경우에 따라 main process에 도움을 줄 수 있는 보조 역할의 container를 더해서 운영해야 할 수도 있다.
  - 이번에도 마찬가지로 literal value를 사용하여 ConfigMap을 생성한다.

  ```bash
  $ kubectl create configmap color --from-literal=color=red
  ```

  - Deployment를 생성한다.

  ```bash
  $ kubectl apply -f https://k8s.io/examples/deployments/deployment-with-configmap-two-containers.yaml
  ```

  - 위에서 Deployment를 생성할 때 사용한 `deployments/deployment-with-configmap-two-containers.yaml` 파일은 아래와 같다.
    - `nginx`와 `alpine`이라는 두 개의 container를 사용하는 Pod들을 생성한다.
    - 두 개의 container는 의사소통을 위해 `emptyDir` volume을 공유한다.
    - `nginx` container는 Nginx web server를 실행시키며, `emptyDir`의 mount 경로는 `/usr/share/nginx/html`이다.
    - `alpine` container는 ConfigMap의 내용을 기반으로하는 HTML 파일을 작성하는 역할을 하며, `emptyDir`의 mount 경로는 `/pod-data`이다.

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: configmap-two-containers
    labels:
      app.kubernetes.io/name: configmap-two-containers
  spec:
    replicas: 3
    selector:
      matchLabels:
        app.kubernetes.io/name: configmap-two-containers
    template:
      metadata:
        labels:
          app.kubernetes.io/name: configmap-two-containers
      spec:
        volumes:
          - name: shared-data
            emptyDir: {}
          - name: config-volume
            configMap:
              name: color
        containers:
          - name: nginx
            image: nginx
            volumeMounts:
              - name: shared-data
                mountPath: /usr/share/nginx/html
          - name: alpine
            image: alpine:3
            volumeMounts:
              - name: shared-data
                mountPath: /pod-data
              - name: config-volume
                mountPath: /etc/config
            command:
              - /bin/sh
              - -c
              - while true; do echo "$(date) My preferred color is $(cat /etc/config/color)" > /pod-data/index.html;
                sleep 10; done;
  ```

  - Pod의 상태를 확인한다.
    - Pod당 2개의 container를 실행하여 READY가 2/2인 것을 확인할 수 있다.

  ```bash
  $ kubectl get pods --selector=app.kubernetes.io/name=configmap-two-contain
  
  # output
  NAME                                        READY   STATUS    RESTARTS   AGE
  configmap-two-containers-565fb6d4f4-2xhxf   2/2     Running   0          20s
  configmap-two-containers-565fb6d4f4-g5v4j   2/2     Running   0          20s
  configmap-two-containers-565fb6d4f4-mzsmf   2/2     Running   0          20s
  ```

  - Deployment를 expose한다.
    - Nginx의 기본 port인 80 port를 target port로 설정한다.

  ```bash
  $ kubectl expose deployment configmap-two-containers --name=configmap-service --port=8080 --target-port=80
  ```

  - Port를 forward 시킨다.
    - `kubectl port-forward`를 통해 local의 port를 Pod로 forward시킬 수 있다.

  ```bash
  $ kubectl port-forward service/configmap-service 8080:8080
  ```

  - Service에 접근해본다.

  ```bash
  $ curl http://localhost:8080
  
  # output
  Wed Nov 27 04:45:34 UTC 2024 My preferred color is red
  ```

  - ConfigMap을 수정한다.
    - `data.color`의 값을 red에서 blue로 수정한다.

  ```bash
  $ kubectl edit configmap color
  ```

  - 잠시 기다린 뒤 다시 Service에 접근해본다.
    - blue로 변경된 것을 확인할 수 있다.

  ```bash
  $ curl http://localhost:8080
  
  # output
  Wed Nov 27 06:12:56 UTC 2024 My preferred color is blue
  ```



- Sidecar container를 실행하는 Pod의 ConfigMap을 수정하여 설정 변경하기

  - Sidecar container
    - 메인 container의 기능을 향상시키거나 확장하기 위해 사용하는 container를 sidecar container라고 부른다.
    - 일반적으로 메인 container에 logging, monitoring, security, data synchronization 등의 기능을 제공한다.
    - Multi container의 일종이며 Kubernetes 환경에서 실제 이용되는 많은 수의 multi container Pod가 이런 패턴을 따르고 있다.
    - Sidecar container는 개념적으로 Init Container이기 때문에, main container가 실행되기 전에 sidecar container가 실행된다는 것이 보장된다.

  - Deployment를 생성한다.
    - 이번에는 이전에 생성한 후 수정한 `color` ConfigMap을 그대로 사용한다.

  ```bash
  $ kubectl apply -f https://k8s.io/examples/deployments/deployment-with-configmap-and-sidecar-container.yaml
  ```

  - 위에서 Deployment를 생성할 때 사용한 `deployments/deployment-with-configmap-and-sidecar-container.yaml`은 아래와 같다.
    - 이번에는 `alpine` container가 sidecar container가 된다.
    - `alpine` container는 `nginx` container가 사용할 HTML file을 생성하는 역할을 한다.
    - `alpine` container는 Init Container이므로 `nginx`보다 먼저 실행된다는 것이 보장되고, 따라서 `nginx` container가 실행될 때, `alpine` container가 생성한 HTML file이 이미 존재하게 된다.
    - Main container와 sidecar container는 이번에도 `emptyDir` volume을 공유하고, mount 경로는 이전과 동일하다.

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: configmap-sidecar-container
    labels:
      app.kubernetes.io/name: configmap-sidecar-container
  spec:
    replicas: 3
    selector:
      matchLabels:
        app.kubernetes.io/name: configmap-sidecar-container
    template:
      metadata:
        labels:
          app.kubernetes.io/name: configmap-sidecar-container
      spec:
        volumes:
          - name: shared-data
            emptyDir: {}
          - name: config-volume
            configMap:
              name: color
        containers:
          - name: nginx
            image: nginx
            volumeMounts:
              - name: shared-data
                mountPath: /usr/share/nginx/html
        initContainers:
          - name: alpine
            image: alpine:3
            restartPolicy: Always
            volumeMounts:
              - name: shared-data
                mountPath: /pod-data
              - name: config-volume
                mountPath: /etc/config
            command:
              - /bin/sh
              - -c
              - while true; do echo "$(date) My preferred color is $(cat /etc/config/color)" > /pod-data/index.html;
                sleep 10; done;
  ```

  - Pod의 상태를 확인한다.

  ```bash
  $ kubectl get pods --selector=app.kubernetes.io/name=configmap-sidecar-container
  ```

  - Deployment를 expose한다.

  ```bash
  $ kubectl expose deployment configmap-sidecar-container --name=configmap-sidecar-service --port=8081 --target-port=80
  ```

  - Port를 forward한다.

  ```bash
  $ kubectl port-forward service/configmap-sidecar-service 8081:8081
  ```

  - Service에 접근해본다.

  ```bash
  $ curl http://localhost:8081
  
  # output
  Sat Feb 17 13:09:05 UTC 2024 My preferred color is blue
  ```

  - ConfigMap을 수정한다.
    - `data.color`를 blue에서 green으로 변경한다.

  ```bash
  $ kubectl edit configmap color
  ```

  - 잠시 기다린 뒤 다시 Service에 접근해본다.
    - green으로 변경된 것을 확인할 수 있다.

  ```bash
  $ curl http://localhost:8081
  
  # output
  Wed Nov 27 06:31:02 UTC 2024 My preferred color is green
  ```



- Volume으로 mount된 immutable한 ConfigMap을 통해 설정 변경하기

  - Immutable ConfigMap
    - Immutable ConfigMap은 고정적이고, 시간의 흘러도 변경될 가능성이 없는 설정을 위해 사용된다.
    - ConfigMap을 immutable하게 만들면, kubelet이 변경 사항을 지속적으로 확인하지 않아도 되므로 성능이 향상될 수 있다.
  - Immutable ConfigMap을 변경하는 방법들
    - ConfigMap의 이름을 변경한 후 실행중인 Pod가 새로운 이름의 ConfigMap을 참조하도록 변경한다.
    - 이전 값을 사용하는 Pod를 실행했던 모든 Node를 교체한다.
    - 이전에 ConfigMap을 load했던 모든 Node에서 kubelet을 재실행한다.
  - Immutable ConfigMap을 생성한다.

  ```bash
  $ kubectl apply -f https://k8s.io/examples/configmap/immutable-configmap.yaml
  ```

  - 위에서 ConfigMap을 생성할 때 사용한 `configmap/immutable-configmap.yaml`의 내용은 아래와 같다.
    - Immutable ConfigMap을 생성한다.
  
  ```yaml
  apiVersion: v1
  data:
    company_name: "ACME, Inc." # existing fictional company name
  kind: ConfigMap
  immutable: true
  metadata:
    name: company-name-20150801
  ```
  
  - Deployment를 추가로 생성한다.
  
  ```bash
  $ kubectl apply -f https://k8s.io/examples/deployments/deployment-with-immutable-configmap-as-volume.yaml
  ```
  
    - 위 Deployment를 생성할 때 사용한 `deployments/deployment-with-immutable-configmap-as-volume.yaml`의 내용은 아래와 같다.
      - 위에서 생성한 `company-name-20150801` immutable ConfigMap을 Pod의 container에 volume으로 mount한다.
  
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: immutable-configmap-volume
    labels:
      app.kubernetes.io/name: immutable-configmap-volume
  spec:
    replicas: 3
    selector:
      matchLabels:
        app.kubernetes.io/name: immutable-configmap-volume
    template:
      metadata:
        labels:
          app.kubernetes.io/name: immutable-configmap-volume
      spec:
        containers:
          - name: alpine
            image: alpine:3
            command:
              - /bin/sh
              - -c
              - while true; do echo "$(date) The name of the company is $(cat /etc/config/company_name)";
                sleep 10; done;
            ports:
              - containerPort: 80
            volumeMounts:
              - name: config-volume
                mountPath: /etc/config
        volumes:
          - name: config-volume
            configMap:
              name: company-name-20150801
  ```
  
    - Pod의 상태를 확인한다.
  
  ```bash
  $ kubectl get pods --selector=app.kubernetes.io/name=immutable-configmap-volume
  ```
  
    - Pod의 log를 확인한다.
      - Pod의 container는 ConfigMap에 정의된 data를 참조하여, 해당 data를 출력한다.
  
  ```bash
  $ kubectl logs deployments/immutable-configmap-volume
  
  # output
  Found 3 pods, using pod/immutable-configmap-volume-78b6fbff95-5gsfh
  Thu Nov 28 00:30:08 UTC 2024 The name of the company is ACME, Inc.
  Thu Nov 28 00:30:18 UTC 2024 The name of the company is ACME, Inc.
  Thu Nov 28 00:30:28 UTC 2024 The name of the company is ACME, Inc.
  ```
  
    - ConfigMap이 immutable이라면 값을 바꿀 수 없다.
      - ConfigMap을 immutable하게 생성한 경우, mutable하게 변경하거나, `data` 혹은 `binaryData` field의 내용을 변경할 수 없다.
      - Immutable한 ConfigMap을 사용하는 Pod의 동작을 변경하려면, 새로운 immutable ConfigMap을 생성하고, 새로 생성한 immutable ConfigMap을 참조하도록 Deployment를 수정해야한다.
  
  
    - 새로운 immutable ConfigMap 생성하기
  
  ```bash
  $ kubectl apply -f https://k8s.io/examples/configmap/new-immutable-configmap.yaml
  ```
  
    - 위에서 ConfigMap을 생성할 때 사용한 `configmap/new-immutable-configmap.yaml`의 내용은 아래와 같다.
  
  ```yaml
  apiVersion: v1
  data:
    company_name: "Fiktivesunternehmen GmbH" # new fictional company name
  kind: ConfigMap
  immutable: true
  metadata:
    name: company-name-20240312
  ```
  
    - Deployment가 새로 생성한 ConfigMap을 참조하도록 수정한다.
      - `spec.template.spec.volumes.configMap.name`의 값을 company-name-20150801에서 company-name-20240312으로 변경한다.
  
  ```bash
  $ kubectl edit deployment immutable-configmap-volume
  ```
  
    - Pod의 상태를 확인한다.
      - 새로 생성된 모든 Pod들이 정상적으로 실행되는지 확인한다.
  
  ```bash
  $ kubectl get pods --selector=app.kubernetes.io/name=immutable-configmap-volume
  ```
  
    - Pod의 로그를 확인한다.
      - 변경된 것을 확인할 수 있다.
  
  ```bash
  $ kubectl logs deployment/immutable-configmap-volume
  
  # output
  Found 3 pods, using pod/immutable-configmap-volume-5fdb88fcc8-n5jx4
  Thu Nov 28 00:45:38 UTC 2024 The name of the company is Fiktivesunternehmen GmbH
  Thu Nov 28 00:45:48 UTC 2024 The name of the company is Fiktivesunternehmen GmbH
  Thu Nov 28 00:45:58 UTC 2024 The name of the company is Fiktivesunternehmen GmbH
  ```
  
    - 기존에 사용하던 immutable ConfigMap을 삭제한다.
  
  ```bash
  $ kubectl delete configmap company-name-20150801
  ```







## ConfigMap을 사용하여 Redis 설정해보기

- ConfigMap 생성하기

  - Configuration block을 비워둔 채로 아래와 같이 ConfigMap 생성을 위한 yaml 파일을 작성한다.

  ```yaml
  # example-redis-config.yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: example-redis-config
  data:
    redis-config: ""
  ```

  - 위 파일을 사용하여 ConfigMap을 생성한다.

  ```bash
  $ kubectl apply -f ./example-redis-config.yaml
  ```



- Redis Pod 생성하기

  - Redis Pod를 생성한다.

  ```bash
  $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
  ```

  - 위에서 Redis Pod를 생성할 때 사용한 `pods/config/redis-pod.yaml`의 내용은 아래와 같다.
    - `config`라는 이름의 volume을 생성한다.
    - `spec.volumes[1].configMap.items[0]`를 보면 `example-redis-config`라는 ConfigMap에서 `redis-config`라는 key에 해당하는 값을 가져온다는 것을 확인할 수 있다.
    - 이 key에 해당하는 값이 파일로 변환되어 `config` 볼륨에 저장되며, 파일 이름은 `redis.conf`이다.
    - `spec.containers[0].volumeMounts[1]`를 보면 `config`라는 이름의 volume이 container의 `/redis-master` 경로에 mount된다는 것을 확인할 수 있다.

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: redis
  spec:
    containers:
    - name: redis
      image: redis:5.0.4
      command:
        - redis-server
        - "/redis-master/redis.conf"
      env:
      - name: MASTER
        value: "true"
      ports:
      - containerPort: 6379
      resources:
        limits:
          cpu: "0.1"
      volumeMounts:
      - mountPath: /redis-master-data
        name: data
      - mountPath: /redis-master
        name: config
    volumes:
      - name: data
        emptyDir: {}
      - name: config
        configMap:
          name: example-redis-config
          items:
          - key: redis-config
            path: redis.conf
  ```



- Redis의 현재 설정 확인하기

  - `redis-cli`에 연결한다.

  ```bash
  $ kubectl exec -it redis -- redis-cli
  ```

  - `redis-cli`에서 `maxmemory` 값을 확인한다.

  ```bash
  127.0.0.1:6379> CONFIG GET maxmemory
  
  # output
  1) "maxmemory"
  2) "0"
  ```

  - `redis-cli`에서 `maxmemory-policy`를 확인한다.

  ```bash
  127.0.0.1:6379> CONFIG GET maxmemory-policy
  
  # output
  1) "maxmemory-policy"
  2) "noeviction"
  ```



- ConfigMap 수정하기

  - 원래 비어 있던 `redis-config`에 아래와 같은 내용을 추가한다.

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: example-redis-config
  data:
    redis-config: |
      maxmemory 2mb
      maxmemory-policy allkeys-lru
  ```

  - ConfigMap을 수정한다.
    - 위에서 수정한 yaml 파일의 내용이 ConfigMap에 반영되도록 한다.

  ```bash
  $ kubectl apply -f example-redis-config.yaml
  ```

  - 변경 사항이 적용됐는지 확인한다.

  ```bash
  $ kubectl describe configmap/example-redis-config
  
  # output
  Name:         example-redis-config
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>
  
  Data
  ====
  redis-config:
  ----
  maxmemory 2mb
  maxmemory-policy allkeys-lru
  ```



- Redis 설정이 변경되었는지 확인하기

  - 확인하기

  ```bash
  $ kubectl exec -it redis -- redis-cli
  
  127.0.0.1:6379> CONFIG GET maxmemory
  # output
  1) "maxmemory"
  2) "0"
  
  127.0.0.1:6379> CONFIG GET maxmemory-policy
  # output
  1) "maxmemory-policy"
  2) "noeviction"
  ```

  - 변하지 않은 것을 확인할 수 있다.

    - 이는 변경된 ConfigMap을 반영하기 위해서 Pod를 재실행해야 하는데, 재실행하지 않았기 때문이다.

    - 아래와 같이 기존의 Pod를 삭제하고, 다시 생성한다.

  ```bash
  $ kubectl delete pod redis
  $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
  ```

  - Redis 설정을 다시 확인한다.
    - 정상적으로 변경된 것을 확인할 수 있다.

  ```bash
  $ kubectl exec -it redis -- redis-cli
  
  127.0.0.1:6379> CONFIG GET maxmemory
  # output
  1) "maxmemory"
  2) "2097152"
  
  127.0.0.1:6379> CONFIG GET maxmemory-policy
  # output
  1) "maxmemory-policy"
  2) "allkeys-lru"
  ```





# Load balancer 사용하기

- Kubernetes Service의 type을 `LoadBalancer`로 설정할 경우, load balancer가 필요하다.

  - 일반적으로 AWS 등의 cloud provider가 제공하는 load balancer를 사용한다.
  - 그러나, 아래에서는 cloud 환경이 아닌 local mahcine으로 테스트를 진행하므로, 별도의 load balancer가 필요하다.
  - minikube의 경우, addon 중에 metallb를 사용하면 load balancer를 사용할 수 있다.
    - 아래와 같이 metallb addon을 활성화한다.

  ```bash
  $ minikube addons enable metallb
  ```

  - metallb 관련 Pod들이 정상적으로 생성 됐는지 확인한다.
    - metallb 관련 Pod들은 `default` namespaec가 아닌 `metallb-system` namespace에 생성된다.
    - 따라서 아래와 같이 `-n` option으로 `metallb-system` namespace를 지정해줘야한다.

  ```bash
  $ kubectl get pods -n metallb-system
  ```

  - metallb 관련 ConfigMap이 생성됐는지 확인한다.
    - 기본적으로 `config`라는 ConfigMap이 생성된다.

  ```bash
  $ kubectl get configmap -n metalb-system
  ```



- Deployment 생성하기

  - Deployment 생성을 위한 yaml file
    - Replica는 5로 설정하여 총 5개의 Pod를 생성한다.
    - 8080 port로 요청을 보낸다.

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      app.kubernetes.io/name: load-balancer-example
    name: hello-world
  spec:
    replicas: 5
    selector:
      matchLabels:
        app.kubernetes.io/name: load-balancer-example
    template:
      metadata:
        labels:
          app.kubernetes.io/name: load-balancer-example
      spec:
        containers:
        - image: gcr.io/google-samples/hello-app:2.0
          name: hello-world
          ports:
          - containerPort: 8080
  ```

  - Deployment 생성

  ```bash
  $ kubectl apply -f https://k8s.io/examples/service/load-balancer-example.yaml
  ```

  - Deployment를 확인한다.

  ```bash
  $ kubectl get deployments hello-world
  $ kubectl describe deployments hello-world
  ```

  - ReplicaSet을 확인한다.

  ```bash
  $ kubectl get replicasets
  $ kubectl describe replicasets
  ```



- Service 생성하기

  - Service를 생성한다.
    - Service의 type은 `LoadBalancer`로 설정한다.

  ```bash
  $ kubectl expose deployment hello-world --type=LoadBalancer --name=my-service
  ```

  - Service의 정보를 확인한다.
    - 아래와 같이 `EXTERNAL-IP`의 값이 `<pending>`인걸 확인할 수 있는데, 이는 load balancer가 없기 때문이다.

  ```bash
  $ kubectl get services my-service
  
  # output
  NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)    AGE
  my-service   LoadBalancer   10.3.245.137   <pending>   		8080/TCP 
  ```

  - Service에 대한 상세 정보를 확인한다.
    - Load balancer가 설정된 상태면 `LoadBalancer Ingress` 항목도 표시되는데, 현재는 load balancer가 없기에 표시되지 않는다.
    - 아래에서 `Endpoints` 항목을 보면 Service가 여러 개의 endpoint들을 가진 것을 볼 수 있다.
    - 이는 application을 실행하는 Pod들의 내부 IP이다.

  ```bash
  $ kubectl describe services my-service
  
  # output
  Name:           my-service
  Namespace:      default
  Labels:         app.kubernetes.io/name=load-balancer-example
  Annotations:    <none>
  Selector:       app.kubernetes.io/name=load-balancer-example
  Type:           LoadBalancer
  IP:             10.3.245.137
  Port:           <unset> 8080/TCP
  NodePort:       <unset> 32377/TCP
  Endpoints:      10.0.0.6:8080,10.0.1.6:8080,10.0.1.7:8080 + 2 more...
  Session Affinity:   None
  Events:         <none>
  ```

  - Pod들의 내부 IP 확인하기
    - `IP` 항복에서 Service 상세 정보의 `Endpoints` 항목에서 봤던 IP들을 확인할 수 있다.

  ```bash
  $ kubectl get pods --output=wide
  
  # output
  NAME                         ...  IP         NODE
  hello-world-2895499144-1jaz9 ...  10.0.1.6   gke-cluster-1-default-pool-e0b8d269-1afc
  hello-world-2895499144-2e5uh ...  10.0.1.8   gke-cluster-1-default-pool-e0b8d269-1afc
  hello-world-2895499144-9m4h1 ...  10.0.0.6   gke-cluster-1-default-pool-e0b8d269-5v7a
  hello-world-2895499144-o4z13 ...  10.0.1.7   gke-cluster-1-default-pool-e0b8d269-1afc
  hello-world-2895499144-segjf ...  10.0.2.5   gke-cluster-1-default-pool-e0b8d269-cpuc
  ```



- Load balancer 설정하기

  - Service가 사용할 수 있도록 metallb의 ConfigMap을 수정한다.
  - metallb addon을 enable하면, `metallb-system` namespace에 `config`라는 이름의 ConfigMap이 생성된다.

  ```bash
  $ kubectl get configmap -n metallb-system
  
  # output
  Name:         config
  Namespace:    metallb-system
  Labels:       <none>
  Annotations:  <none>
  
  Data
  ====
  config:
  ----
  address-pools:
  - name: default
    protocol: layer2
    addresses:
    - -
  
  
  BinaryData
  ====
  
  Events:  <none>
  ```

  - metallb가 자동으로 생성한 `config` ConfigMap을 아래와 같이 수정한다.
    - Address pool을 지정한다.
    - `192.168.49.100-192.168.49.100`와 같이 지정하면 오직 `192.168.49.100`만 사용할 수 있는 것과 동일하다.
    - `192.168.49.100`로 설정하는 이유는, 우선 minikube는 기본적으로 192.168.49.0/24 서브넷을 사용한다.
    - 따라서 이 IP범위 내의 주소(192.168.49.1부터 192.168.49.254)를 사용 가능하다.
    - 그러므로 address pool 역시 192.168.49.1부터 192.168.49.254 사이의 구간으로 지정할 수 있지만, `192.168.49.1-192.168.49.10`정도의 낮은 구간은 minikube가 자체적으로 사용할 수 있으므로 충돌을 피하기 위해 `192.168.49.100-192.168.49.200` 정도의 구간을 선택한다.
    - 그런데, 현재는 오직 my-service 하나만을 실행할 것이므로 굳이 넓은 구간을 지정하지 않고, `192.168.49.100` 하나만 사용하도록 한다.

  ```bash
  $ cat <<EOF | kubectl apply -f -
  apiVersion: v1
  kind: ConfigMap
  metadata:
    namespace: metallb-system
    name: config
  data:
    config: |
      address-pools:
      - name: default
        protocol: layer2
        addresses:
        - 192.168.49.100-192.168.49.100
  EOF
  ```

  - ConfigMap이 잘 수정되었는지 확인한다.

  ```bash
  $ kubectl describe configmap config -n metallb-system
  
  # output
  Name:         config
  Namespace:    metallb-system
  Labels:       <none>
  Annotations:  <none>
  
  Data
  ====
  config:
  ----
  address-pools:
  - name: default
    protocol: layer2
    addresses:
    - 192.168.49.100-192.168.49.100
  
  
  BinaryData
  ====
  
  Events:  <none>
  ```

  - Service를 확인한다.
    - `EXTERNAL-IP`의 값이 우리가 위에서 설정한 `192.168.49.100`로 변경된 것을 확인할 수 있다.
    - 또한 `LoadBalancer Ingress` 항목도 표시되며, `Events` 항목에서 할당 내역을 볼 수 있다.

  ```bash
  $ kubectl get services my-service
  # output
  NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)    AGE
  my-service   LoadBalancer   10.3.245.137   192.168.49.100   8080/TCP   54s
  
  $ kubectl describe services my-service
  # output
  # ...
  LoadBalancer Ingress:     192.168.49.100
  # ...
  Events:
    Type    Reason        Age                    From                Message
    ----    ------        ----                   ----                -------
    Normal  IPAllocated   9m40s                  metallb-controller  Assigned IP "192.168.49.100"
    Normal  nodeAssigned  8m27s (x2 over 9m40s)  metallb-speaker     announcing from node "minikube-m02"
  ```



- Application에 접근해보기

  - `LoadBalancer Ingress`에 있는 IP로 application에 요청을 보내본다.

  ```bash
  $ curl 192.168.49.100:8080
  ```

  - 만약 Docker Desktop을 container driver로 사용하여 minikube를 실행한다면, minikube tunnel이 필요하다.
    - 이는 Docker Desktop의 container가 host computer로부터 격리되어 있기 때문이다.
    - `Hostname`을 통해 어느 Pod에서 요청을 처리했는지를 확인할 수 있다.
    - 요청을 반복적으로 보내보면 각기 다른 Pod로 요청이 분산되는 것을 확인할 수 있다.

  ```bash
  # 먼저 아래 명령어를 실행한다.
  $ minikube service test-deployment --url
  
  # 위 명령어를 실행하면 ttp://127.0.0.1:51082와 같은 url이 나올텐데, 위 명령어를 실행한 창을 그대로 띄워둔 상태에서 출력된 url로 요청을 보내면 된다.
  $ curl ttp://127.0.0.1:51082
  
  # output
  Hello, world!
  Version: 2.0.0
  Hostname: hello-world-4ttdc175u3-m2b4t
  ```








# StatefulSets

- StatefulSets
  - StatefulSet은 데이터를 보존해야 하는 저장소나 잘 변하지 않고, 고유한 network가 필요할 때 유용하게 사용할 수 있다.
    - Deployment와 유사하게 StatefulSet은 Pod들을 관리하는 역할을 한다.
    - 그러나 Deployment와는 달리 StatefulSet은 각 Pod에 대한 고정된 identity를 함께 관리한다.
    - Pod들은 같은 spec으로 생성되지만, 각기 다른 identity를 가지고 있으므로 서로 교체될 수 없다.
  - Stateful application(DB 등) 또는 distributed system(Elasticsearch 등)을 위해 사용한다.



- StatefulSet과 headless Service의 관계
  - StatefulSet
    - StatefulSet은 각 Pod에 고유한 이름과 순서를 보장하는데, 이를 활용하려면 각 Pod의 DNS name이 고유해야한다.
    - 이를 지원하기 위해 headless Service가 사용된다.
  - Headless Service
    - 일반적인 Service는 cluster 내부에서 load balancing을 제공하고, 모든 Pod를 단일 DNS name으로 노출한다.
    - 반면 headless Service는 load balancing을 하지 않고, DNS record만 생성하여 개별 Pod에 직접 접근할 수 있게 해준다.
  - 아래와 같은 이유로 둘을 함께 사용한다.
    - StatefulSet의 Pod는 고유한 이름과 함께 headless Service를 통해 DNS 이름이 생성되는데, 이를 통해 특정 Pod에 직접 접근이 가능해진다.
    - 또한 headless Service는 load balancing 없이 Pod의 개별 DNS entry를 제공하므로, Pod간 직접 통신이 필요한 application에서 유용하다.
    - 그리고 StatefulSet의 주요 특징 중 하나는 각 Pod의 안정적인 network ID인데(예를 들어, pod-0이 삭제되었다 재생성되더라도 동일한 DNS name이 유지된다), headless Service가 이를 지원한다.



- StatefulSet과 headless Service 생성하기

  - 생성을 위한 yaml 파일
    - 윗 부분은 `nginx`라는 이름의 Headless Service를 생성하는 것으로 `spec.clusterIP`를 보면 `None`으로 설정된 것을 볼 수 있다.
    - 아래는 `web`이라는 이름의 StatefulSet을 생성하는 것이다.
    - `spec.volumeClaimTempates`를 설정하면 PersistentVolumeClaim이 생성된다.

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx
    labels:
      app: nginx
  spec:
    ports:
    - port: 80
      name: web
    clusterIP: None
    selector:
      app: nginx
  ---
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: web
  spec:
    serviceName: "nginx"
    replicas: 2
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: registry.k8s.io/nginx-slim:0.21
          ports:
          - containerPort: 80
            name: web
          volumeMounts:
          - name: www
            mountPath: /usr/share/nginx/html
    volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
  ```

  - Service와 StatefulSet을 생성하기 전에, 생성 과정을 보기 위해 별도의 terminal에서 아래 명령어를 수행한다.
    - `--watch` flag는 상태 변경을 실시간으로 보여준다.
    - `-l`은 label로 Pod를 filtering한다.
    - 결과적으로 `nginx`라는 label을 가진 Pod들의 상태 변경 과정이 출력된다.

  ```bash
  $ kubectl get pods --watch -l app=nginx
  ```

  - Service와 StatefulSet을 생성한다.

  ```bash
  $ kubectl apply -f https://k8s.io/examples/application/web/web.yaml
  ```

  - 위에서 생성한 `nginx` Service와 `web` StatefulSet을 확인한다.
    - `nginx` Service의 `CLUSTER-IP`의 값이 None인 것을 볼 수 있다.

  ```bash
  $ kubectl get service nginx
  # output
  NAME      TYPE         CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
  nginx     ClusterIP    None         <none>        80/TCP    12s
  
  $ kubectl get statefulset web
  # output
  NAME   READY   AGE
  web    2/2     37s
  ```

  - 위에서 실행했던 `kubectl get pods --watch -l app=nginx`의 출력은 아래와 같다.
    - n개의 replica를 가진 StatefulSet에 대해 Pod들이 배포되면, {0..n-1}의 순서로 순차적으로 생성된다.
    - 아래 출력 결과를 보면 `web-0` Pod가 `Running` 상태가 되기 전까지 `web-1`이 실행되지 않는 것을 확인할 수 있다.

  ```bash
  NAME      READY     STATUS    RESTARTS   AGE
  web-0     0/1       Pending   0          0s
  web-0     0/1       Pending   0         0s
  web-0     0/1       ContainerCreating   0         0s
  web-0     1/1       Running   0         19s
  web-1     0/1       Pending   0         0s
  web-1     0/1       Pending   0         0s
  web-1     0/1       ContainerCreating   0         0s
  web-1     1/1       Running   0         18s
  ```



- StatefulSet 내부의 Pod

  - StatefulSet 내부의 Pod들은 고유한 ordinal index와 고정된 network identity를 가진다.
  - Pod의 ordinal index 확인하기
    - Pod의 이름은 `<stateful_name>-<ordinal_index>`의 형식으로 생성된다.

  ```bash
  $ kubectl get pods -l app=nginx
  
  # output
  NAME      READY     STATUS    RESTARTS   AGE
  web-0     1/1       Running   0          1m
  web-1     1/1       Running   0          1m
  ```

  - 각 Pod는 ordinal index에 기반한 고정된 hostname을 가지고 있다.
    - 아래 명령어를 실행하면 각 Pod에 `hostname` 명령어를 실행해 볼 수 있다.

  ```bash
  $ for i in 0 1; do kubectl exec "web-$i" -- sh -c 'hostname'; done
  
  # output
  web-0
  web-1
  ```

  - 각 Pod의 IP를 확인하기 위해  `busybox` container를 실행한다.
    - `busybox`는 소형 Linux 기반 도구 모음으로 네트워크 테스트 및 디버깅에 자주 사용된다.
    - `busybox`에는 `dnsutils` package가 포함되어 있으며, 해당 package의  `nslookup` 명령어를 통해 도메인 이름을 IP 주소로 변환하거나, IP 주소를 기반으로 도메인 이름을 확인할 수 있다.

  ```bash
  $ kubectl run -it --image busybox:1.28 dns-test --restart=Never --rm
  ```

  - Pod의 hostname에 `nslookup` 명령어를 실행하여 각 Pod의 cluster 내부 DNS address를 확인할 수 있다.

  ```sh
  $ nslookup web-0.nginx
  # output
  Server:    10.0.0.10
  Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local
  
  Name:      web-0.nginx
  Address 1: 10.244.1.6
  
  
  $ nslookup web-1.nginx
  # output
  Server:    10.0.0.10
  Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local
  
  Name:      web-1.nginx
  Address 1: 10.244.2.6
  ```

  - StatefulSet에 속한 Pod에서 아래와 같은 것들은 변하지 않는다.
    - Ordinals
    - Hostname
    - SRV record
    - A record
  - 그러나 Pod와 연관된 IP 주소는 변경될 수도 있다.
    - 따라서 application이 StatefulSet 내의 특정 Pod의 IP 주소로 직접 연결하도록 해선 안된다.
    - 단, hostname은 변하지 않으므로 Pod의 hostname으로는 직접 연결해도 된다.
  - StatefulSet 내의 Pod들 찾기
    - 만약 StatefulSet 내의 Pod들 중 active Pod들을 찾고자 한다면, headless Service의 CNAME(예시의 경우 `nginx.default.svc.cluster.local`)에 query를 전송해야 한다. CNAME과 관련된 SRV record는 오직 `Running` 혹은 `Ready` 상태인 Pod들만 포함하고 있다.
    - 만약 application에 connection의 활성화 상태와 준비 상태를 확인할 수 있는 logic이 구현되어 있고, Pod가 안정된 상태라면, SRV record(예시의 경우 `web-0.nginx.default.svc.cluster.local`, `web-1.nginx.default.svc.cluster.local`)를 사용할 수 도 있다.



- Stable storage에 쓰기

  - Stateful set을 생성할 때 함께 생성된 PVC 정보를 확인한다.
    - StatefulSet controller는 두 개의 PV에 binding된 두 개의 PVC를 생성했다.
    - 현재 사용하는 cluster에서는 PV를 dynamic하게 provisioning하도록 설정했기 때문에, PV는 자동으로 생성되고 binding된다.

  ```bash
  $ kubectl get pvc -l app=nginx
  
  # output
  NAME        STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
  www-web-0   Bound     pvc-15c268c7-b507-11e6-932f-42010a800002   1Gi        RWO           48s
  www-web-1   Bound     pvc-15c79307-b507-11e6-932f-42010a800002   1Gi        RWO           48s
  ```

  - Nginx에 요청 보내보기
    - 아래와 같이 요청을 보내면 Nginx web server는 `/usr/share/nginx/html/index.html` 파일을 읽어서 반환하고, 없으면 404 Not Found를 반환한다.
    - 혹은 `/usr/share/nginx/html` 경로 혹은 `/usr/share/nginx/html/index.html` 파일에 대한 접근 권한이 없으면 403 Forbidden을 반환한다.

  ```bash
  $ kubectl exec -i -t "web-0" -- curl http://localhost/
  
  # output
  <html>
  <head><title>403 Forbidden</title></head>
  <body bgcolor="white">
  <center><h1>403 Forbidden</h1></center>  
  <hr><center>nginx/1.13.3</center>        
  </body>
  </html>
  ```

  - `index.html` 파일에 쓰기
    - Nginx webserver는 기본적으로 `/usr/share/nginx/html/index.html` 파일을 serve한다.
    - 아래와 같이 `/usr/share/nginx/html/index.html` 파일을 작성한 후 다시 요청을 보낸다.
    - 위에서 Statefulset을 생성할 때 설정한 `spec.template.spec.volumeMounts` 값으로 인해 `/usr/share/nginx/html` directory가 PV에 의해 지원된다는 것이 보장되므로, directory 생성 없이 바로 file을 생성할 수 있다.

  ```bash
  # /usr/share/nginx/html/index.html 파일에 hostname을 작성한다.
  $ kubectl exec "web-0" -- sh -c 'echo "$(hostname)" > /usr/share/nginx/html/index.html'
  
  # 다시 요청을 보낸다.
  kubectl exec -i -t "web-0" -- curl http://localhost/
  
  # output
  web-0
  ```

  - Pod 삭제하기
    - Pod를 모두 삭제하고 새로운 Pod가 다시 생성될 때 까지 기다린다.

  ```bash
  $ kubectl delete pod -l app=nginx
  ```

  - Pod 삭제 이후에도 PV에 작성한 값이 남아 있는지 확인하기
    - 이전 Pod에서 PV에 작성한 내용이 삭제되지 않고 그대로 남아있는 것을 확인할 수 있다.
    - 이는 PVC와 연결된 PV가 `spec.template.spec.volumeMounts`에 설정된 경로로 다시 mount되었기 때문이다.

  ```bash
  $ kubectl exec -it "web-0" -- curl http://localhost/
  
  # output
  web-0
  ```





## Scaling a StatefulSet

- StatefuleSet의 scaling
  - StatefulSet을 scaling한다는 것은 replica의 개수를 늘리거나 줄이는 것을 의미한다.
  - Scaling을 위해서는 `replica` field의 값을 수정해야한다.
    - `kubectl scale` 혹은 `kubectl patch`로 가능하다.



- Scaling up

  - `kubectl scale` 명령어를 통해 scaling up 한다.
    - `sts`는 StatefulSet의 약어이다.
    - 아래 명령어는 `web`이라는 StatefulSet의 replica를 5로 변경한다는 의미이다.

  ```bash
  $ kubectl scale sts web --replicas=5
  ```

  - 확인
    - 기존의 `web-0`, `web-1` 외에 `web-[2..4]`가 추가된 것을 볼 수 있다.

  ```bash
  $ kubectl get pod -l app=nginx
  ```



- Scaling down

  - `kubectl patch` 명령어를 통해 scaling down한다.

  ```bash
  $ kubectl patch sts web -p '{"spec":{"replicas":3}}'
  ```

  - 확인
    - 새로 추가된 `web-[2..4]` 중 `web-3`과 `web-4`가 삭제된 것을 볼 수 있다.

  ```bash
  $ kubectl get pod -l app=nginx
  ```

  - Pod의 종료 순서
    - Control plane은 Pod를 한 번에 하나씩만 제거한다.
    - 이 때 Pod의 ordinal index에 따라 역순으로 제거한다(즉, index가 큰 Pod부터 제거한다).
    - 위 예시의 경우 `web-3`과 `web-4`중 `web-4`가 먼저 제거된다.
  - PVC 확인하기
    - 아래와 같이 PVC를 확인해보면, Pod가 제거된 이후에도 PVC는 여전히 남아있는 것을 확인할 수 있다.

  ```bash
  $ kubectl get pvc -l app=nginx
  
  # output
  NAME        STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
  www-web-0   Bound     pvc-15c268c7-b507-11e6-932f-42010a800002   1Gi        RWO           13h
  www-web-1   Bound     pvc-15c79307-b507-11e6-932f-42010a800002   1Gi        RWO           13h
  www-web-2   Bound     pvc-e1125b27-b508-11e6-932f-42010a800002   1Gi        RWO           13h
  www-web-3   Bound     pvc-e1176df6-b508-11e6-932f-42010a800002   1Gi        RWO           13h
  www-web-4   Bound     pvc-e11bb5f8-b508-11e6-932f-42010a800002   1Gi        RWO           13h
  ```







 ## StatefulSet update하기

- StatefulSet controller는 자동 update를 지원한다.
  - 이는 container image, resource request 혹은 limit, label, 그리고 Pod의 annotation을 수정할 때 사용할 수 있다.
  - 자동 update시에 사용하는 전략은 `spec.updateStratgy`값에 따라 결정된다.
    - `RollingUpdate`와 `OnDelete`라는 두 가지 전략을 사용할 수 있다.



- RollingUpdate

  - StatefulSet 내부의 모든 Pod들을 ordinal order의 역순으로 update한다.
  - StatefulSet을 update하기 전에, Pod들을 watch하기

  ```bash
  $ kubectl get pod -l app=nginx --watch
  ```

  - `web` StatefulSet을 update하기
    - 위에서 watch를 실행한 terminal과 다른 terminal에서 아래 명령어를 실행한다.
    - `web` StatefulSet에 있는 container들의 image를 변경한다.

  ```bash
  $ kubectl patch statefulset web --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"registry.k8s.io/nginx-slim:0.24"}]'
  ```

  - `--watch`를 실행한 terminal을 확인하면 아래와 같은 결과를 볼 수 있다.
    - 각 Pod들이 ordinal index의 역순으로(2에서 0으로) update된다.
    - StatefulSet controller는 하나의 Pod를 종료하고, 다시 생성하여 새로운 Pod가 실행 상태가 되면, 다음 Pod를 종료하고 다시 생성하는 과정을 반복한다.
    - 만약 update 전에 에러가 발생할 경우, 실패한 Pod는 기존 version으로 복원한다.
    - 반면에 update 후에 에러가 발생할 경우, 실패한 Pod는 update된 version으로 복원한다.

  ```bash
  NAME      READY     STATUS    RESTARTS   AGE
  web-0     1/1       Running   0          7m
  web-1     1/1       Running   0          7m
  web-2     1/1       Running   0          8m
  web-2     1/1       Terminating   0         8m
  web-2     1/1       Terminating   0         8m
  web-2     0/1       Terminating   0         8m
  web-2     0/1       Terminating   0         8m
  web-2     0/1       Terminating   0         8m
  web-2     0/1       Terminating   0         8m
  web-2     0/1       Pending   0         0s
  web-2     0/1       Pending   0         0s
  web-2     0/1       ContainerCreating   0         0s
  web-2     1/1       Running   0         19s
  web-1     1/1       Terminating   0         8m
  web-1     0/1       Terminating   0         8m
  web-1     0/1       Terminating   0         8m
  web-1     0/1       Terminating   0         8m
  web-1     0/1       Pending   0         0s
  web-1     0/1       Pending   0         0s
  web-1     0/1       ContainerCreating   0         0s
  web-1     1/1       Running   0         6s
  web-0     1/1       Terminating   0         7m
  web-0     1/1       Terminating   0         7m
  web-0     0/1       Terminating   0         7m
  web-0     0/1       Terminating   0         7m
  web-0     0/1       Terminating   0         7m
  web-0     0/1       Terminating   0         7m
  web-0     0/1       Pending   0         0s
  web-0     0/1       Pending   0         0s
  web-0     0/1       ContainerCreating   0         0s
  web-0     1/1       Running   0         10s
  ```

  - Image가 정상적으로 update 되었는지 확인한다.
    - 모든 Pod에서 image update가 성공적으로 완료된 것을 볼 수 있다.

  ```bash
  $ for p in 0 1 2; do kubectl get pod "web-$p" --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'; echo; done
  
  # output
  registry.k8s.io/nginx-slim:0.24
  registry.k8s.io/nginx-slim:0.24
  registry.k8s.io/nginx-slim:0.24
  ```



- Staging an update

  - `RollingUpdate` 전략을 사용하는 StatefulSet의 경우 update를 parition으로 나눠서 실행할 수 있다.
    - `.spec.updateStrategy.rollingUpdate.partition`의 값을 정의하면 된다.
  - StatefulSet에서 update를 staging할 수 있다.
    - 이 방식은 StatefulSet에 대한 Pod template을 변경하는 동안 StatefulSet의 기존 Pod를 변경하지 않는다.
    - 이후 직접 혹은 자동적으로 trigger를 작동시켜 update를 수행할 수 있다.
    - `.spec.updateStrategy.rollingUpdate.partition`를 통해 정의할 수 있다.
  - 먼저 partition을 설정한다.
    - 아래에서 `partition`의 값은 update가 적용될 ordinal을 설정하는 것으로, 현재 StatefulSet 내부의 마지막 ordinal 보다 큰 값을 설정해야 한다.

  ```bash
  $ kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":3}}}}'
  ```

  - 그 후에 container의 image를 변경한다.

  ```bash
  $ kubectl patch statefulset web --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"registry.k8s.io/nginx-slim:0.21"}]'
  ```

  - Pod에 대한 watch를 실행한다.

  ```bash
  $ kubectl get pod -l app=nginx --watch
  ```

  - StatefulSet 내부의 Pod를 삭제한다.

  ```bash
  $ kubectl delete pod web-2
  ```

  - Watch의 결과는 아래와 같다.
    - Pod가 재생성 되었다.

  ```bash
  NAME      READY     STATUS              RESTARTS   AGE
  web-0     1/1       Running             0          4m
  web-1     1/1       Running             0          4m
  web-2     0/1       ContainerCreating   0          11s
  web-2     1/1       Running   			0          18s
  ```

  - Container의 image가 변경되었는지 확인한다.
    - Image가 변경되지 않았다.
    - 이는 `web-2` Pod의 ordinal이 `partition`에 설정된 값보다 작기 때문이다.

  ```bash
  $ kubectl get pod web-2 --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'
  
  # output
  registry.k8s.io/nginx-slim:0.24
  ```



- Rolling out a canary

  - Pod에 대한 watch를 실행한다.

  ```bash
  $ kubectl get pod -l app=nginx --watch
  ```

  - 위에서 stage했던 update를 canary rollout을 실행할 것이다.
    - `partition`의 값을 점차 감소시키는 방법으로 실행한다.

  ```bash
  $ kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":2}}}}'
  ```

  - Watch의 결과는 아래와 같다.
    - `partition`의 값이 2로 감소하면서 `web-2` Pod가 update 대상이 됐고, 위에서 staging되어 있던 update가 `web-2` Pod에 적용됐다.

  ```bash
  NAME    READY   STATUS    RESTARTS   AGE
  web-0   1/1     Running   0          25m
  web-1   1/1     Running   0          25m
  web-2   1/1     Running   0          7m41s
  web-2   1/1     Terminating   0          7m49s
  web-2   0/1     Completed     0          7m50s
  web-2   0/1     Completed     0          7m50s
  web-2   0/1     Completed     0          7m50s
  web-2   0/1     Pending       0          0s
  web-2   0/1     Pending       0          0s
  web-2   0/1     ContainerCreating   0          0s
  web-2   1/1     Running             0          1s
  ```

  - Image가 update 되었는지 확인한다.
    - 정상적으로 변경된 것을 확인할 수 있다.

  ```bash
  $ kubectl get pod web-2 --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'
  
  # output
  registry.k8s.io/nginx-slim:0.21
  ```

  - `partition`에 속하지 않는 Pod를 종료시켜본다.

  ```bash
  $ kubectl delete pod web-1
  ```

  - 재실행된 Pod의 image가 update 되었는지 확인한다.
    - 변경되지 않은 것을 확인할 수 있다.
    - 이처럼, `partition` 보다 낮은 ordinal을 가지고 있는 Pod들은 삭제되거나 종료되더라도, 원래 설정으로 복원된다.

  ```bash
  $ kubectl get pod web-1 --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'
  
  # output
  registry.k8s.io/nginx-slim:0.24
  ```



- Phased roll out

  - Partitioned rolling update를 활용하면 linear, geometric, exponential roll out 등의 phased roll out을 수행할 수 있다.
    - 예를 들어 0~9까지 9개의 Pod가 있다고 가정해보자.
    - 먼저 `partition`의 값을 `10`으로 설정한 후 update를 수행한다.
    - 이 경우, `partition`의 값이 모든 Pod의 ordinal 보다 크기에 어떤 Pod도 update되지 않는다.
    - 먼저 1개의 Pod에만 적용하기로 결정이 되어 `partition`의 값을 9로 변경한다.
    - 9번 Pod에만 update가 적용되고, 9번 Pod를 대상으로 정상 동작하는지 확인한다.
    - 정상 동작하는 것이 확인되어 `partition`의 값을 7로 변경하고 7~8번 Pod가 update된 후 정상 동작 여부를 확인한다.
    - 정상 동작하면 `partition`의 값을 4로 변경하고 4~6번  Pod가 update된 후 정상 동작 여부를 확인한다.
    - 정상 동작하면 `partition`의 값을 0으로 변경하고 0~3번  Pod가 update된 후 정상 동작 여부를 확인한다.

  - 아래와 같이 `partition`의 값을 수정한다.
    - 예시의 경우 0으로 설정한다.

  ```bash
  $ kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":0}}}}'
  ```

  - 모든 Pod가 update되었는지 확인한다.

  ```bash
  $ for p in 0 1 2; do kubectl get pod "web-$p" --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'; echo; done
  
  # output
  registry.k8s.io/nginx-slim:0.21
  registry.k8s.io/nginx-slim:0.21
  registry.k8s.io/nginx-slim:0.21
  ```



- OnDelete 전략

  - `spec.template.updateStrategy.type`을 `OnDelete`로 설정하면 사용할 수 있다.

  ```bash
  $ kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"OnDelete"}}}'
  ```

  - 이 전략은 `spec.template`이 수정되더라도 Pod를 자동으로 update하지 않는다.
    - Update를 적용하려면 Pod를 수동 혹은 자동으로 삭제해야 한다.





## StatefulSet 삭제하기

- StatefulSet 삭제하기
  - StatefulSet은 non-cascading과 cascading 삭제를 모두 지원한다.
  - Non-cascading delete
    - StatefulSet의 Pod는 StatefulSet이 삭제되더라도 삭제되지 않는다.
  - Cascading delete
    - StatefulSet이 삭제되면 Pod도 함께 삭제된다.



- Non-cascading delete

  - `kubectl delete` command를 `--cascade=orphan` parameter와 함께 사용한다.

  ```bash
  $ kubectl delete statefulset web --cascade=orphan
  ```

  - Pod의 상태를 확인한다.
    - StatefulSet `web`이 삭제되었음에도, Pod는 남아있는 것을 확인할 수 있다.

  ```bash
  $ kubectl get pods -l app=nginx
  
  # output
  NAME      READY     STATUS    RESTARTS   AGE
  web-0     1/1       Running   0          6m
  web-1     1/1       Running   0          7m
  web-2     1/1       Running   0          5m
  ```

  - Pod는 직접 삭제해야한다.

  ```bash
  $ kubectl delete pod web-0
  ```

  - Pod가 삭제되었는지 확인한다.
    - StatefulSet이 삭제되었으므로, `web-0` Pod는 재실행되지 않는다.

  ```bash
  $ kubectl get pods -l app=nginx 
  
  # output
  NAME      READY     STATUS    RESTARTS   AGE
  web-1     1/1       Running   0          10m
  web-2     1/1       Running   0          7m
  ```

  - `web` StatefulSet을 다시 생성한다.
    - `web/web.yaml`에 설정된 `replicas`의 값이 2이므로, `web-0` Pod가 생성되고, `web-2` Pod는 종료된다.

  ```bash
  $ kubectl apply -f https://k8s.io/examples/application/web/web.yaml
  ```

  - PV가 여전히 유지되는지 확인한다.
    - StatefulSet과 `web-0` Pod를 삭제했음에도, 여전히 PV가 유지되고 있는 것을 확인할 수 있다.
    - 이는 StatefulSet이 삭제되더라도 PV는 삭제되지 않기 때문이다.
    - StatefulSet이 재생성되고, `web-0` Pod가 재실행되면서 PV역시 remount된다.

  ```bash
  $ kubectl exec -i -t "web-0" -- curl http://localhost/
  
  # output
  web-0
  ```



- Cascading delete

  - Watch를 실행한다.

  ```bash
  $ kubectl get pods --watch -l app=nginx
  ```

  - Cascading delete를 실행한다.
    - 이번에는 `--cascade=orphan` parameter를 주지 않는다.

  ```bash
  $ kubectl delete statefulset web
  ```

  - Watch의 결과는 아래와 같다.
    - Scaling down할 때와 마찬가지로 한 번에 하나의 Pod만 종료되며, ordinal index의 역순으로 종료된다.

  ```bash
  NAME    READY   STATUS    RESTARTS   AGE
  web-0   1/1     Running   0          4m53s
  web-1   1/1     Running   0          62m
  web-1   1/1     Terminating   0          63m
  web-0   1/1     Terminating   0          5m29s
  web-0   0/1     Completed     0          5m29s
  web-1   0/1     Completed     0          63m
  web-0   0/1     Completed     0          5m30s
  web-1   0/1     Completed     0          63m
  web-0   0/1     Completed     0          5m30s
  web-1   0/1     Completed     0          63m
  ```

  - Service 삭제하기
    - Cascading delete는 StatefulSet과 그 Pod들을 함께 삭제하지만, StatefulSet과 관련된 Service를 함께 삭제하지는 않는다.
    - 따라서 Service는 수동으로 삭제해야한다.

  ```bash
  $ kubectl delete service nginx
  ```

  - StatefulSet과 headless Service를 다시 생성한다.

  ```bash
  $ kubectl apply -f https://k8s.io/examples/application/web/web.yaml
  ```

  - PV가 여전히 적용되는지 확인한다.
    - 적용되는 것을 확인할 수 있다.

  ```bash
  $ for i in 0 1; do kubectl exec -i -t "web-$i" -- curl http://localhost/; done
  
  # output
  web-0
  web-1
  ```

  - 다시 StatefulSet과 Service를 삭제한다.

  ```bash
  $ kubectl delete service nginx
  $ kubectl delete statefulset web
  ```





## Pod management policy

- Pod management policy
  - 분산 시스템 중에는 StatefulSet의 순서 보장이 불필요하거나 순서 보장이 되면 안 되는 경우가 있을 수 있다.
  - Kubernetes는 이런 경우를 위해 Pod management policy를 설정할 수 있게 해준다.
    - `OrderReady`(default)나 `Parallel` 둘 중 하나의 값으로 설정할 수 있다.



- OrderReady Pod management
  - 위에서 살펴본 방식으로 동작한다.
  - 정확한 순서로 update를 진행해야 하는 application에 사용한다.



- `Parallel Pod management`

  - 모든 Pod를 병렬적으로 처리하도록 설정한다.
    - Pod들은 순차적으로 처리되지 않으며, 따라서 하나의 Pod가 `Running`, `Ready` 상태가 될 때까지 다른 Pod가 기다리지 않는다.
    - `Parallel` Pod management option은 오직 scaling 작업의 동작에만 영향을 미치며 update에는 영향을 미치지 않는다.
    - 따라서 순서에 따라 roll out을 수행하는 것이 가능하다.

  - StatefulSet을 생성을 위한 `web/web-parallel.yaml` 파일은 아래와 같다.
    - `spec.podManagementPolicy`를 `Parallel`로 설정한다.

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx
    labels:
      app: nginx
  spec:
    ports:
    - port: 80
      name: web
    clusterIP: None
    selector:
      app: nginx
  ---
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: web
  spec:
    serviceName: "nginx"
    podManagementPolicy: "Parallel"
    replicas: 2
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: registry.k8s.io/nginx-slim:0.24
          ports:
          - containerPort: 80
            name: web
          volumeMounts:
          - name: www
            mountPath: /usr/share/nginx/html
    volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
  ```

  - Watch를 실행한다.

  ```bash
  $ kubectl get pod -l app=nginx --watch
  ```

  - StatefulSet과 Service를 생성한다.

  ```bash
  $ $ kubectl apply -f https://k8s.io/examples/application/web/web-parallel.yaml
  ```

  - StatefulSet의 replica를 변경한다.

  ```bash
  $ kubectl scale statefulset/web --replicas=5
  ```

  - Watch의 결과는 아래와 같다.
    - 하나의 Pod가 완전히 생성될 때까지 기다리지 않고, 세 개의 추가 Pod를 병렬적으로 생성한다.

  ```bash
  NAME    READY   STATUS    RESTARTS   AGE
  web-0   0/1     Pending   0          0s
  web-1   0/1     Pending   0          0s
  web-0   0/1     Pending   0          0s
  web-1   0/1     Pending   0          0s
  web-0   0/1     ContainerCreating   0          0s
  web-1   0/1     ContainerCreating   0          0s
  web-0   1/1     Running             0          1s
  web-1   1/1     Running             0          1s
  web-2   0/1     Pending             0          0s
  web-2   0/1     Pending             0          0s
  web-4   0/1     Pending             0          0s
  web-3   0/1     Pending             0          0s
  web-4   0/1     Pending             0          0s
  web-3   0/1     Pending             0          0s
  web-2   0/1     ContainerCreating   0          0s
  web-3   0/1     ContainerCreating   0          0s
  web-4   0/1     ContainerCreating   0          0s
  web-4   1/1     Running             0          1s
  web-2   1/1     Running             0          2s
  web-3   1/1     Running             0          2s
  ```

