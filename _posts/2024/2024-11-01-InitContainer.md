---
title:  "Kubernetes Init Container & PodInitializing"
excerpt: "Kubernetes 이야기"

toc: true
toc_sticky: true

categories:
  - Kubernetes
tags:
  - Init Container
  - k8s
  - Kubernetes 이야기
---



# [Kubernetes Init Container & PodInitializing](https://waspro.tistory.com/643)

Kubernetes는 배포 최소단위로 Pod를 사용한다. Pod는 하나 이상의 Container를 포함하는 배포 단위이다.

이때, 컨테이너는 여러 형태로 존재할 수 있는데,

기동 시점에 처리하고 종료되고 Init Container

실제 업무를 처리하는 Runtime Container

보조 역할로써 배포되는 SideCar Container 등이 존재한다.

우리는 일반적으로 생각하는 Runtime Container만 다루곤 하는데, 때로는 다른 형태의 컨테이너를 이해하고 있어야 실제 장애에 대응할 수 있는 상황이 주어지기도 한다.

예를 들어 다음과 같은 상황을 살펴보도록 하자.

앞서 다음 포스팅에서 트러블슈팅을 진행하는 몇가지 방법에 대해 알아보았다.

 

[(참조) Kubernetes - 장애 발생 시 진단 Process 정의](https://waspro.tistory.com/563?category=831751)

 

이를 준수하여 트러블 슈팅을 충분히 진행할 수 있지만, 다음과 같은 상황에서는 대처할 수 없다.

먼저 Pod를 기동하였더니, Pod는 PodInitializing 상태이고, init:0/2가 반복적으로 출력되는 상태를 감지하였다. 로그를 살펴보았더니, 특별한 로그가 없었으며, Event 역시 특별히 진단할 트러블슈팅 포인트를 찾지 못했다.

이때 의심해 봐야 할 포인트는?

바로 Init Container이다.

## Init Container (초기화 컨테이너)

초기화 컨테이너는 Pod의 Runtime Container가 실행되기 전에 실행되는 컨테이너이며, 이미지에는 포함되지 않는 (Dockerfile에 정의되어 있지 않은) 유틸리티 또는 설정 스크립트 등을 포함할 수 있다.

Pod는 앱들을 실행하는 다수의 컨테이너를 포함할 수 있고, 또한 Runtime Container 일명 App Container가 실행되기 전 동작되는 하나 이상의 초기화 컨테이너도 포함할 수 있다.
초기화 컨테이너는 다음 초기화 컨테이너가 시작되기 전에 성공적으로 완료되어야 한다.

예를 들어 Init Container가 1, 2 Runtime Container가 1이 존재하고 각 실행 순서가 Init 1 - Init 2 - Runtime 1의 순서로 처리될 경우 Init 1이 완료되어야 Init 2가 실행되고 Init 2가 완료되어야 Runtime 1이 실행된다는 의미이다.
만약 초기화 컨테이너가 실패한다면, 쿠버네티스는 초기화 컨테이너가 성공할 때까지 파드를 반복적으로 재시작한다. 그러나, 만약 파드의 restartPolicy 를 절대 하지 않음(Never)으로 설정했다면, 파드는 재시작되지 않고 종료된다.

## Init Container 예시

다음은 busybox container를 deploy하는 pod yaml 파일이다.

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

일반적으로 사용하는 spec.containers가 runtime container이며, spec.initContainers가 바로 초기화 컨테이너이다.

위 내용은 init-myservice → init-mydb → myapp-container 순으로 container가 기동되고 처리된다.

기동 되는 순서를 차례대로 살펴보자면, 다음과 같은 순서로 기동된다.

```
[root@kubemaster ~]# kubectl get pods -w
NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   0/1     Pending   0          0s
myapp-pod   0/1     Pending   0          0s
myapp-pod   0/1     Init:0/2   0          1s
myapp-pod   0/1     Init:0/2   0          1s
myapp-pod   0/1     Init:1/2   0          8s
myapp-pod   0/1     PodInitializing   0          9s
myapp-pod   1/1     Running           0          10s
```

Pod는 Pending 상태로 시작하여 Init Container 1 (init-myservice)가 기동되는 Init:0/2, Init Container 2 (init-mydb)가 기동되는 Init:1/2, 그리고 Runtime Container (myapp-container)가 기동되는 PodInitializing 마지막으로 모든 Pod가 기동 완료된 상태인 Running 상태로 변경된다.

이때 initContainer의 상태에 문제가 발생할 경우 Pod는 기동되지 못하고 init container가 정상적으로 기동될 때까지(?? init container는 정상 처리되면 처리 후 종료되는 컨테이너이니 표현이 적당하지 않을 수 있습니다.) 반복적으로 retry한다.

이와 같은 현상이 발생했을때, 추적하는 방법에 대해 알아보도록 하자.

## Init Container & PodInitializing Restarting TroubleShooting

초기화 컨테이너에 의해 발생가능한 에러 포인트는 두가지가 있다.

### 1) 초기화 컨테이너 자체의 문제

초기화 컨테이너가 처리되지 못할 경우 다음과 같은 STATUS를 나타낸다.

```
[root@kubemaster ~]# kubectl get pods -w
NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   0/1     Pending   0          0s
myapp-pod   0/1     Pending   0          0s
myapp-pod   0/1     Init:0/3   0          1s
myapp-pod   0/1     Init:0/3   0          2s
myapp-pod   0/1     Init:1/3   0          6s
myapp-pod   0/1     Init:2/3   0          7s
myapp-pod   0/1     Init:Error   0          8s
myapp-pod   0/1     Init:Error   1          9s
myapp-pod   0/1     Init:CrashLoopBackOff   1          22s
myapp-pod   0/1     Init:Error              2          23s
myapp-pod   0/1     Init:CrashLoopBackOff   2          37s
myapp-pod   0/1     Init:Error              3          52s
myapp-pod   0/1     Init:CrashLoopBackOff   3          65s
...
...
```

위와 같이 반복적으로 Init:~~ 상태가 나타날 경우 Init Container 기동에 문제가 발생한 것이다.

이때는 다음과 같이 진단을 진행한다.

먼저 기존처럼 pod name으로 log를 확인해서는 안된다.

```
[root@kubemaster ~]# kubectl logs -f myapp-pod
Error from server (BadRequest): container "myapp-container" in pod "myapp-pod" is waiting to start: PodInitializing
[root@kubemaster ~]#
```

위와 같이 로그를 확인해 보면 단순이 Pod가 Initializing되기를 대기하고 있다는 메시지만 나타날 뿐이다.

이때는 다음과 같이 확인 후 init container의 상태를 진단해야 한다.

먼저 describe 정보를 확인한다.

```
[root@kubemaster ~]# kubectl describe pod myapp-pod
Name:         myapp-pod
Namespace:    default
Priority:     0
Node:         kubeworker1/192.168.56.103
Start Time:   Sat, 26 Sep 2020 23:30:55 +0900
Labels:       app=myapp
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"myapp"},"name":"myapp-pod","namespace":"default"},"spec":{"c...
Status:       Pending
IP:           10.233.124.123
IPs:
  IP:  10.233.124.123
Init Containers:
  init-myservice:
    Container ID:  docker://d82413032fa576e8500fe8312d78725d699e406872ddd054ceea41d7b0744166
    Image:         busybox:1.28
    Image ID:      docker-pullable://busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sat, 26 Sep 2020 23:30:56 +0900
      Finished:     Sat, 26 Sep 2020 23:31:00 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-thvqh (ro)
  init-mydb:
    Container ID:  docker://14153e276af21275b58522b26f302882622b7511ff65905cf85e579cf5204e04
    Image:         busybox:1.28
    Image ID:      docker-pullable://busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sat, 26 Sep 2020 23:31:01 +0900
      Finished:     Sat, 26 Sep 2020 23:31:01 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-thvqh (ro)
  init-failed-test:
    Container ID:  docker://873a63fddc3599e975128ae9783311a46e4375941b9686d4d7e2ebeb04020894
    Image:         busybox:1.28
    Image ID:      docker-pullable://busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      hello.sh
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    127
      Started:      Sat, 26 Sep 2020 23:34:06 +0900
      Finished:     Sat, 26 Sep 2020 23:34:06 +0900
    Ready:          False
    Restart Count:  5
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-thvqh (ro)
Containers:
  myapp-container:
    Container ID:  
    Image:         busybox:1.28
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The app is running! && sleep 3600
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-thvqh (ro)
Conditions:
  Type              Status
  Initialized       False 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-thvqh:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-thvqh
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                    From                  Message
  ----     ------     ----                   ----                  -------
  Normal   Scheduled  <unknown>              default-scheduler     Successfully assigned default/myapp-pod to kubeworker1
  Normal   Pulled     4m20s                  kubelet, kubeworker1  Container image "busybox:1.28" already present on machine
  Normal   Created    4m20s                  kubelet, kubeworker1  Created container init-myservice
  Normal   Started    4m20s                  kubelet, kubeworker1  Started container init-myservice
  Normal   Pulled     4m15s                  kubelet, kubeworker1  Container image "busybox:1.28" already present on machine
  Normal   Created    4m15s                  kubelet, kubeworker1  Created container init-mydb
  Normal   Started    4m15s                  kubelet, kubeworker1  Started container init-mydb
  Normal   Pulled     3m30s (x4 over 4m14s)  kubelet, kubeworker1  Container image "busybox:1.28" already present on machine
  Normal   Created    3m30s (x4 over 4m14s)  kubelet, kubeworker1  Created container init-failed-test
  Normal   Started    3m30s (x4 over 4m14s)  kubelet, kubeworker1  Started container init-failed-test
  Warning  BackOff    2m52s (x7 over 4m12s)  kubelet, kubeworker1  Back-off restarting failed container
[root@kubemaster ~]# 
```

Describe 정보를 확인해 보면 Init Container가 존재하는 것을 확인할 수 있다.

현재 해당 컨테이너는 init-myservice, init-mydb, init-failed-test 컨테이너가 존재하는 것을 확인할 수 있다.



![img](\assets\images\InitContainer.png)



각각의 State와 Reason을 알아보면, init-myservice, init-mydb은 Terminated/Completed이지만, init-failed-test는 Waiting/CrashLoopBackOff 상태인 것을 알 수 있다. 물론 myapp-container는 Waiting/PodInitializing 상태이다.

kubectl 명령어를 통해서도 확인이 가능하다.

(kubectl get pod myapp-pod --template '{{.status.initContainerStatuses}}')

```
[root@kubemaster ~]# kubectl get pod myapp-pod --template '{{.status.initContainerStatuses}}'
[map[containerID:docker://d82413032fa576e8500fe8312d78725d699e406872ddd054ceea41d7b0744166 image:busybox:1.28 imageID:docker-pullable://busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47 lastState:map[] name:init-myservice ready:true restartCount:0 state:map[terminated:map[containerID:docker://d82413032fa576e8500fe8312d78725d699e406872ddd054ceea41d7b0744166 exitCode:0 finishedAt:2020-09-26T14:31:00Z reason:Completed startedAt:2020-09-26T14:30:56Z]]] map[containerID:docker://14153e276af21275b58522b26f302882622b7511ff65905cf85e579cf5204e04 image:busybox:1.28 imageID:docker-pullable://busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47 lastState:map[] name:init-mydb ready:true restartCount:0 state:map[terminated:map[containerID:docker://14153e276af21275b58522b26f302882622b7511ff65905cf85e579cf5204e04 exitCode:0 finishedAt:2020-09-26T14:31:01Z reason:Completed startedAt:2020-09-26T14:31:01Z]]] map[containerID:docker://b5c52e752b0c79b31cd5c0e6111b844e286d5a968dfac055f2968594a78ca783 image:busybox:1.28 imageID:docker-pullable://busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47 lastState:map[terminated:map[containerID:docker://b5c52e752b0c79b31cd5c0e6111b844e286d5a968dfac055f2968594a78ca783 exitCode:127 finishedAt:2020-09-26T14:52:02Z reason:Error startedAt:2020-09-26T14:52:02Z]] name:init-failed-test ready:false restartCount:9 state:map[waiting:map[message:back-off 5m0s restarting failed container=init-failed-test pod=myapp-pod_default(9363416d-1076-4a1f-8130-b03a35b51e0d) reason:CrashLoopBackOff]]]]
[root@kubemaster ~]#
```

init-failed-test container의 상태가 ready:false 상태이고 restartCount가 9회 중인것을 확인할 수 있다.

로그는 다음과 같이 확인한다.

```
[root@kubemaster ~]# kubectl logs -f myapp-pod -c init-myservice
Server:    169.254.25.10
Address 1: 169.254.25.10

Name:      myservice.default.svc.cluster.local
Address 1: 218.38.137.27
[root@kubemaster ~]# kubectl logs -f myapp-pod -c init-mydb
Server:    169.254.25.10
Address 1: 169.254.25.10

Name:      mydb.default.svc.cluster.local
Address 1: 218.38.137.27
[root@kubemaster ~]# kubectl logs -f myapp-pod -c init-failed-test
sh: hello.sh: not found
[root@kubemaster ~]# 
```

위와 같이 container의 로그를 확인해보면, init-failed-test container에 hello.sh을 실행하고자 하였으나 shell file이 존재하지 않아 문제가 발생한 것을 확인할 수 있다.

따라서 해당 init container를 정의한 yaml 파일을 수정하거나, 불필요할 경우 해당 container를 제거하는 방법 등을 진단을 내세울 수 있다.

### 2) Init Container의 결과로 인한 Pod 기동 문제

예를 들자면 다음과 같은 경우가 발생할 수 있다.

Init Container에서 특정 디렉토리의 권한을 변경하여 Pod에서 사용할 PV Local 볼륨으로 적용하고자 한다. 이때, 권한 변경이 잘못된 계정으로 수행되어 Background로 실패가 발생하였거나, 또는 권한이 부족하거나, 계정이 잘못되었거나, 여러가지 사유로 인해 실패가 발생할 수 있다.

역시 이 경우에는 진단의 절차가 매우 복잡해진다.

이 경우에는 실제 해당 Init Container가 수행한 동작 결과를 하나하나 따져봐야한다.

예를 들어

네트워크 구조를 변경했다면, 왜 변경했을까?

디렉토리 권한이나, 파일 권한을 변경했다면, 왜 변경했을까?

이미지 구조를 변경하지 않고 원본을 유지하기 위해 init container를 사용했다면, 적용 방법을 바꿔볼까?

런타임 컨테이너의 기동 조건을 명시할수도 있지 않을까? Init Container가 기동되야 Runtime Container가 기동되니까.. 이것 때문에 Pod가 기동이 안되는 건가?

이미지 자체의 보안 취약점을 위해 Init Container를 사용한 건가?

등등....

수많은 경우로 인해 Init Container는 사용될 수 있고 그 용도를 정확히 파악할 필요가 있다.

따라서 Init Container가 없었다면 정상적으로 기동될 수 있는 Runtime Container 상황이었는지 여부를 먼저 점검하고, 정상적으로 동작한다면, Init Container를 다시한번 하나씩 적용하여 검토하는 과정이 필요할 것이다.

## 결론

Init Container는 이미지를 변경하지 않고도 이미지를 구성할 수 있도록 도와주는 유용한 Pod 구성 방법이다.

몇 가지 주의점을 잘 따르고 적용한다면, Pod를 보다 유연하게 관리하고 커스터마이징 해 나갈 수 있을 것이다.
