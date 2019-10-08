# 通过 Object yaml文件 创建部署

## 一、创建.yaml文件

在 `~/deployment` 目录下创建 `nginx-test.yaml` 文件,[参考官方示例](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#describing-a-kubernetes-object)

此object 将创建 nginx-test-deployment 的部署

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-test-deployment
spec:
  selector:
    matchLabels:
      app: nginx-test # 声明 pod 的 labels
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx-test # 声明 pod 的 labels 必须和 spec.selector.matchLabels.app 对应
    spec:
      containers:
      - name: nginx
        image: nginx:1.15
        ports:
        - containerPort: 80
```

## 二、创建部署和pod

`kubectl apply -f ~/deployment/nginx-test.yaml --record`

输出

`deployment.apps/nginx-test-deployment created`

当然 .yaml 文件可以放在远程云存储或者其他 origin，文件用url地址指向文件

让我们查看一下我们的应用

`kubectl get pods`

输出

```
nginx-test-deployment-59f686fd9b-46pxt   1/1     Running   0          11m
nginx-test-deployment-59f686fd9b-bxwbc   1/1     Running   0          11m
vela-nginx-deployment-6c855ccfc8-cd82d   1/1     Running   0          3h59m
```

让我们通过 labels 查看一下pod详情

`kubectl describe pod -l app=nginx-test`

输出

```yaml
Name:         nginx-test-deployment-59f686fd9b-46pxt
Namespace:    default
Priority:     0
Node:         kubernetes-master/xxx.xxx.xxx.xxx
Start Time:   Tue, 08 Oct 2019 07:55:38 +0000
Labels:       app=nginx-test
              pod-template-hash=59f686fd9b
Annotations:  cni.projectcalico.org/podIP: 192.168.237.18/32
Status:       Running
IP:           192.168.237.18
IPs:
  IP:           192.168.237.18
Controlled By:  ReplicaSet/nginx-test-deployment-59f686fd9b
Containers:
  nginx:
    Container ID:   docker://bc05fbf81a2e2c958fd9aaf980b12b3cd5375e273fa25e2d894339400cfc0d49
    Image:          nginx:1.15
    Image ID:       docker-pullable://nginx@sha256:23b4dcdf0d34d4a129755fc6f52e1c6e23bb34ea011b315d87e193033bcd1b68
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 08 Oct 2019 07:55:40 +0000
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
  Type    Reason     Age        From                        Message
  ----    ------     ----       ----                        -------
  Normal  Scheduled  <unknown>  default-scheduler           Successfully assigned default/nginx-test-deployment-59f686fd9b-46pxt to kubernetes-master
  Normal  Pulled     12m        kubelet, kubernetes-master  Container image "nginx:1.15" already present on machine
  Normal  Created    12m        kubelet, kubernetes-master  Created container nginx
  Normal  Started    12m        kubelet, kubernetes-master  Started container nginx


Name:         nginx-test-deployment-59f686fd9b-bxwbc
Namespace:    default
Priority:     0
Node:         kubernetes-node1/xxx.xxx.xxx.xxx
Start Time:   Tue, 08 Oct 2019 07:55:38 +0000
Labels:       app=nginx-test
              pod-template-hash=59f686fd9b
Annotations:  cni.projectcalico.org/podIP: 192.168.129.83/32
Status:       Running
IP:           192.168.129.83
IPs:
  IP:           192.168.129.83
Controlled By:  ReplicaSet/nginx-test-deployment-59f686fd9b
Containers:
  nginx:
    Container ID:   docker://41ba3c22b698d3d2065090d3a5173ec22cab240860bd6655c534b35626acb740
    Image:          nginx:1.15
    Image ID:       docker-pullable://nginx@sha256:23b4dcdf0d34d4a129755fc6f52e1c6e23bb34ea011b315d87e193033bcd1b68
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 08 Oct 2019 07:55:40 +0000
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
  Normal  Scheduled  <unknown>  default-scheduler          Successfully assigned default/nginx-test-deployment-59f686fd9b-bxwbc to kubernetes-node1
  Normal  Pulled     12m        kubelet, kubernetes-node1  Container image "nginx:1.15" already present on machine
  Normal  Created    12m        kubelet, kubernetes-node1  Created container nginx
  Normal  Started    12m        kubelet, kubernetes-node1  Started container nginx
```

我们得到两个 pod

## 三、操作应用（更新、横向缩放pod）

我们可以修改.yaml 文件里的

spec.replicas: 副本数量

spec.template.spec.containers.image: 新的docker镜像

然后再次执行 `kubectl apply -f ~/deployment/nginx-test.yaml --record`

输出

`deployment.apps/nginx-test-deployment configured`

或者

通过命令指定新的镜像版本

`kubectl set image deployments/nginx-test-deployment nginx-test-deployment=nginx:1.17`

通过缩放副本数量命令

`kubectl scale deployments/nginx-test-deployment --replicas=1`

建议：通过更改远程.yaml源文件 + ci/cd(jenkins) 操作应用的更新/缩放。

当然 Kubernetes 支持 autoscale 自动横向缩放，请移步相关章节
