# 更新App和回滚App

## 一、查看当前部署、Pod、Image 版本

查看部署

`kubectl get deployment`
 
 输出

 ```
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
vela-nginx-deployment   1/1     1            1           11d
 ```

查看Pod

`kubectl get pods`

输出

```
NAME                                     READY   STATUS    RESTARTS   AGE
vela-nginx-deployment-6c855ccfc8-pmw6d   1/1     Running   0          11d
```

查看 Pod 详情

`kubectl describe pods`

输出

```yaml
Name:         vela-nginx-deployment-6c855ccfc8-pmw6d
Namespace:    default
Priority:     0
Node:         kubernetes-master/xxx.xxx.xxx.xxx
Start Time:   Thu, 26 Sep 2019 07:50:11 +0000
Labels:       app=vela-1.0
              pod-template-hash=6c855ccfc8
              run=vela-nginx-deployment
Annotations:  cni.projectcalico.org/podIP: 192.168.237.13/32
Status:       Running
IP:           192.168.237.13
IPs:
  IP:           192.168.237.13
Controlled By:  ReplicaSet/vela-nginx-deployment-6c855ccfc8
Containers:
  vela-nginx-deployment:
    Container ID:   docker://a2c0cb03efd5449ea9f298724de515920459ad3dc53155d6ae35eda48ba5dbb5
    Image:          nginx:1.16
    Image ID:       docker-pullable://nginx@sha256:0d0af9bc6ca2db780b532a522a885bef7fcaddd52d11817fc4cb6a3ead3eacc0
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 26 Sep 2019 07:50:15 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-wkk9f (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-wkk9f:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-wkk9f
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>
```

我们查看当前Image版本为 1.16

```yaml
Image:          nginx:1.16
```
## 二、更新Image

执行命令更新Image，这个动作是在不宕机的情况下热更新，通过更改image镜像更新应用，生产环境建议通过yaml文件部署更新，请移步yaml篇

`kubectl set image deployments/vela-nginx-deployment vela-nginx-deployment=nginx:1.17`

输出

`deployment.apps/vela-nginx-deployment image updated`

我们通过pod详情查看更新结果、也可以通过watch查看，需要另外开启一个终端tab

`kubectl describe pods` || `watch kubectl describe pods`

更新后的信息

```yaml
Name:         vela-nginx-deployment-5bf77599b9-phtmr
Namespace:    default
Priority:     0
Node:         kubernetes-node1/xxx.xxx.xxx.xxx
Start Time:   Tue, 08 Oct 2019 03:43:54 +0000
Labels:       pod-template-hash=5bf77599b9
              run=vela-nginx-deployment
Annotations:  cni.projectcalico.org/podIP: 192.168.129.80/32
Status:       Running
IP:           192.168.129.80
IPs:
  IP:           192.168.129.80
Controlled By:  ReplicaSet/vela-nginx-deployment-5bf77599b9
Containers:
  vela-nginx-deployment:
    Container ID:   docker://58badc803d6f520991913d2c4b137c8e3063a234a1a58b09c23af6e117ebb2fe
    Image:          nginx:1.17
    Image ID:       docker-pullable://nginx@sha256:aeded0f2a861747f43a01cf1018cf9efe2bdd02afd57d2b11fcc7fcadc16ccd1
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 08 Oct 2019 03:43:57 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-wkk9f (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-wkk9f:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-wkk9f
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                       Message
  ----    ------     ----       ----                       -------
  Normal  Scheduled  <unknown>  default-scheduler          Successfully assigned default/vela-nginx-deployment-5bf77599b9-phtmr to kubernetes-node1
  Normal  Pulling    4m8s       kubelet, kubernetes-node1  Pulling image "nginx:1.17"
  Normal  Pulled     4m7s       kubelet, kubernetes-node1  Successfully pulled image "nginx:1.17"
  Normal  Created    4m7s       kubelet, kubernetes-node1  Created container vela-nginx-deployment
  Normal  Started    4m7s       kubelet, kubernetes-node1  Started container vela-nginx-deployment
```

得到Image版本为1.17

```yaml
Image:          nginx:1.17
```

## 三、回滚上一个部署状态

我们在新开的终端上观察pod详情

`watch kubectl describe pods`

执行回滚操作

`kubectl rollout undo deployment/vela-nginx-deployment`

输出

``

我们查看pod详情 `kubectl describe pods`，或者观察watch窗口中的Image值

```yaml
Name:         vela-nginx-deployment-6c855ccfc8-cd82d
Namespace:    default
Priority:     0
Node:         kubernetes-node1/xxx.xxx.xxx.xxx
Start Time:   Tue, 08 Oct 2019 04:07:24 +0000
Labels:       pod-template-hash=6c855ccfc8
              run=vela-nginx-deployment
Annotations:  cni.projectcalico.org/podIP: 192.168.129.81/32
Status:       Running
IP:           192.168.129.81
IPs:
  IP:           192.168.129.81
Controlled By:  ReplicaSet/vela-nginx-deployment-6c855ccfc8
Containers:
  vela-nginx-deployment:
    Container ID:   docker://8ff6aacb3aa1413b9eeead9be56c0f65f8b0755d8dd75a05becc25a0dc099a98
    Image:          nginx:1.16
    Image ID:       docker-pullable://nginx@sha256:0d0af9bc6ca2db780b532a522a885bef7fcaddd52d11817fc4cb6a3ead3eacc0
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 08 Oct 2019 04:07:25 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-wkk9f (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-wkk9f:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-wkk9f
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                       Message
  ----    ------     ----       ----                       -------
  Normal  Scheduled  <unknown>  default-scheduler          Successfully assigned default/vela-nginx-deployment-6c855ccfc8-cd82d to kubernetes-node1
  Normal  Pulled     76s        kubelet, kubernetes-node1  Container image "nginx:1.16" already present on machine
  Normal  Created    76s        kubelet, kubernetes-node1  Created container vela-nginx-deployment
  Normal  Started    76s        kubelet, kubernetes-node1  Started container vela-nginx-deployment
```

得到 Image 版本回滚到 1.16

```yaml
Image:          nginx:1.16
```

