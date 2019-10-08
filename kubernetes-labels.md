# 标签(Labels) 用法

## 一、根据Label查看 Pod 和 Services

我们执行 `kubectl run` 命令自动创建了一个部署和 `pod`，同时也自动为我们的 `pod` 创建了一个Label

使用 `describe deployment` 通过查看 `deployment` 命令可以查看标签的名称

`kubectl describe deployment`

输出

```yaml
Name:                   vela-nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 26 Sep 2019 07:50:11 +0000
Labels:                 run=vela-nginx-deployment
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=vela-nginx-deployment
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=vela-nginx-deployment
  Containers:
   vela-nginx-deployment:
    Image:        nginx:1.16
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   vela-nginx-deployment-6c855ccfc8 (1/1 replicas created)
Events:          <none>
```

```yaml
Labels: run=vela-nginx-deployment
```
就是kubernetes自动为我们添加的Label，可以使用这个Label 查询 pod 列表,`-l` 是根据 Label 查询的参数

`kubectl get pods -l run=vela-nginx-deployment`

输出

```
NAME                                     READY   STATUS    RESTARTS   AGE
vela-nginx-deployment-6c855ccfc8-pmw6d   1/1     Running   0          20h
```

也可以用相同的操作来列出对应 `Label` 的 `services`

`kubectl get svc -l run=vela-nginx-deployment`

输出

```
NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
vela-nginx-deployment   NodePort   10.109.46.19   <none>        80:31679/TCP   19h
```

## 二、更改Label

你可以导出 pod 名称保存到环境变量

```
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME
```

`Name of the Pod: vela-nginx-deployment-6c855ccfc8-pmw6d`

输出

`kubectl label pod $POD_NAME app=vela-1.0`

或者通过 查询 `pod` 名字

`kubectl get pods`

输出

```
NAME                                     READY   STATUS    RESTARTS   AGE
vela-nginx-deployment-6c855ccfc8-pmw6d   1/1     Running   0          20h
```

`kubectl label pod vela-nginx-deployment-6c855ccfc8-pmw6d app=vela-1.0`

输出

`pod/vela-nginx-deployment-6c855ccfc8-pmw6d labeled`

更改成功，让我们查看一下pod

`kubectl describe pods $POD_NAME`

或者

`kubectl describe pods vela-nginx-deployment-6c855ccfc8-pmw6d`

输出

```yaml
Name:         vela-nginx-deployment-6c855ccfc8-pmw6d
Namespace:    default
Priority:     0
Node:         kubernetes-master/165.227.222.240
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

看到
```yaml
Labels:       app=vela-1.0
              pod-template-hash=6c855ccfc8
              run=vela-nginx-deployment
```
更改 Label 成功

我们可以使用新的 Label: `app=vela-1.0` 查询 pod 列表

`kubectl get pod -l app=vela-1.0`

输出

```
NAME                                     READY   STATUS    RESTARTS   AGE
vela-nginx-deployment-6c855ccfc8-pmw6d   1/1     Running   0          20h
```

接下来我们更改 Services 的 Label，先列出所有 services

`kubectl get svc`

输出

```
NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes              ClusterIP   10.96.0.1      <none>        443/TCP        2d19h
vela-nginx-deployment   NodePort    10.109.46.19   <none>        80:31679/TCP   21h
```

我们查看一下 `Services: vela-nginx-deployment` 的详细信息

`kubectl describe svc vela-nginx-deployment`

输出

```yaml
Name:                     vela-nginx-deployment
Namespace:                default
Labels:                   run=vela-nginx-deployment
Annotations:              <none>
Selector:                 run=vela-nginx-deployment
Type:                     NodePort
IP:                       10.109.46.19
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31679/TCP
Endpoints:                192.168.237.13:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

当前 Services 的 Label 是 run=vela-nginx-deployment

我们更改一下 `app=vela-1.0`

`kubectl label svc vela-nginx-deployment app=vela-1.0`

输出

`service/vela-nginx-deployment labeled`

列出更新后的Services列表

`kubectl get svc -l app=vela-1.0`

输出

```
NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
vela-nginx-deployment   NodePort   10.109.46.19   <none>        80:31679/TCP   21h
```

我们再次查看 `Services: vela-nginx-deployment` 的详情

可以用 Services Name 查看详情

`kubectl describe svc vela-nginx-deployment` 

或者

通过 Label 查看 Services 详情

 `kubectl describe svc -l app=vela-1.0`

输出

```yaml
Name:                     vela-nginx-deployment
Namespace:                default
Labels:                   app=vela-1.0
                          run=vela-nginx-deployment
Annotations:              <none>
Selector:                 run=vela-nginx-deployment
Type:                     NodePort
IP:                       10.109.46.19
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31679/TCP
Endpoints:                192.168.237.13:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

我们看到当前

```yaml
Labels:                   app=vela-1.0
                          run=vela-nginx-deployment
```

Services 的 Label 已经成功修改了
