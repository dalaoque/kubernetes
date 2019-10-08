# 创建一个部署和Pod并且公开一个应用

检查 kubectl 命令是否可用

`kubectl version --short`

`kubectl version --output='yaml'`

`kubectl version -h`

查看是否有节点可用 `STATUS: Ready`

`kubectl get nodes`

```
NAME                STATUS   ROLES    AGE   VERSION
kubernetes-master   Ready    master   42h   v1.16.0
kubernetes-node1    Ready    <none>   41h   v1.16.0
```

## 一、创建一个部署和 pod

当执行以下命令时同时创建 `deployment` 和 `pod`

`vela-nginx-deployment`: 一个部署（deployment）名字

`--port`: pod 暴露docker容器内部 端口 80，此端口很重要，不能重复，需要通过 services 暴露给 kubernetes `外部完成上线，下文会描述services` 如何启动

`kubectl run vela-nginx-deployment --image=nginx:1.16 --port=80`

```
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/vela-nginx-deployment created
```

如上输出：不推荐使用 `kubectl run` 命令创建部署，所以 Production 环境不推荐这种方式部署，为了快速完成 App 在 kubernetes 上的部署，故先采用此方法，后续会补充通过 `yaml` 文件创建 `deployment` 的最佳实践。

查看部署 READY: 1/1

`kubectl get deployment`

```
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
vela-nginx-deployment   1/1     1            1           22m
```

查看 Pod STATUS: Running

`kubectl get pods`

```
NAME                                     READY   STATUS    RESTARTS   AGE
vela-nginx-deployment-6c855ccfc8-pmw6d   1/1     Running   0          11s
```

## 二、访问 pod 应用接口

`kubernetes` 内部的 `pod` 运行在一个私有的、隔离的网络上。默认情况下，它们在同一个 `kubernetes` 集群内的其他 `pod` 和服务中可见，但在该网络之外则不可见。当我们使用 `kubectl` 时，我们通过一个 `api` 端点与我们的应用程序进行交互

新建一个终端Tab，执行命令创建一个代理 `proxy`，可以通过 `control-C` 终止代理服务

```
echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response. Please click the first Terminal Tab\n";
kubectl proxy --port 8001
```

切换第一个终端Tab访问 `api` 代替 `kubectl version`

`curl http://localhost:8001/version`

```
{
  "major": "1",
  "minor": "16",
  "gitVersion": "v1.16.0",
  "gitCommit": "2bd9643cee5b3b3a5ecbd3af49d09018f0773c77",
  "gitTreeState": "clean",
  "buildDate": "2019-09-18T14:27:17Z",
  "goVersion": "go1.12.9",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

`vela-nginx-deployment-6c855ccfc8-pmw6d`: 是根据 `deployment` 创建的 `pod` 副本名字，因为副本可以横向拓展并且可以设置多个 `pod` 副本，所以我们要访问某一个 `pod`，需要制定副本带哈希的 `pod name`，可以通过 `kubectl get pods` 查看 `pod` 列表

通过 `api` 访问 `pod` 应用

```curl http://localhost:8001/api/v1/namespaces/default/pods/vela-nginx-deployment-6c855ccfc8-pmw6d/proxy/```

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

恭喜！ `pod` 创建成功并且正常运行并且能够通过内部 `api` 正常访问

## 三、通过 Services 发现服务并公开应用

`Services` 是 `kubernetes` 提供的针对容器的服务发现与负载均衡机制的资源，并通过 `kube-proxy` 配合 `cloud provider` 来适应不同的应用场景，本文不做展开叙述，请移步Services篇

查看当前所有 services

`kubectl get svc` || `kuberctl get services`

```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   45h
```

已经有了一个 `Services` `kubernetes`，这是 `kubeadm init` 时默认创建的。我们需要创建一个自己的 `services`

首先查看一下 deployment

`kubectl get deployment`

```
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
vela-nginx-deployment   1/1     1            1           75m
```

执行 `kubectl expose` 把 `deployment` 公开

`--port 80`: 是创建 `deployment` 时候 暴露 `pod` 中 `docker` 容器内部的端口 80,上文提到的 `kubectl run`命令

`kubectl expose deployment/vela-nginx-deployment --type="NodePort" --port 80`

`service/vela-nginx-deployment exposed`

再次查看所有 `services`

`kubectl get svc`

```
NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes              ClusterIP   10.96.0.1      <none>        443/TCP        45h
vela-nginx-deployment   NodePort    10.109.46.19   <none>        80:31679/TCP   62s
```

如此我们看到 `services` 列表中多了一个 名字为 `vela-nginx-deployment` 的 `services`

让我们查看一下 `services` `vela-nginx-deployment` 的描述

`kubectl describe svc vela-nginx-deployment` || `kubectl describe services vela-nginx-deployment`

```
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
`NodePort: unset>  31679/TCP` 31679 就是公开此应用的端口号

此时我们可以通过局域网络可以访问`kubernetes-master`的其他计算机访问，如果`kubernetes-master`是部署在服务器上并且拥有公网IP，此时应用就已经上线了，接下来是做域名解析到公网IP上

让我们测试一下 `curl Kubernetes-master节点IP:31679`

`curl xxx.xxx.xxx.xxx:31679`

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

恭喜！我们成功公开我们的部署应用

## 四、动态横向拓展 pod

我们先查看一下所有 pod

`kubectl get pods`

```
NAME                                     READY   STATUS    RESTARTS   AGE
vela-nginx-deployment-6c855ccfc8-pmw6d   1/1     Running   0          25h
```

目前是一个 pod

让我们查看一下 deployment 列表

`kubectl get deployment`

输出

```
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
vela-nginx-deployment   1/1     1            1           25h
```

我们看到 `READY: 1/1` 当前RUNNING状态的pod数量1 / 副本拓展总数量1

让我们查看一下部署 deployment 详情

`kubectl describe deployment vela-nginx-deployment`

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

为我们的 `vela-nginx-deployment` 部署横向拓展4个副本

`kubectl scale deployments/vela-nginx-deployment --replicas=4`

输出

`deployment.apps/vela-nginx-deployment scaled`

让我们再次查看一下 deployment 列表

`kubectl get deployment`

输出

```
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
vela-nginx-deployment   4/4     4            4           25h
```

我们看到 `READY: 4/4` 当前RUNNING状态的pod数量4 / 副本拓展总数量4

让我们查看 `deployment/vela-nginx-deployment` 详情

`kubectl describe deployment vela-nginx-deployment`

输出

```yaml
Name:                   vela-nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 26 Sep 2019 07:50:11 +0000
Labels:                 run=vela-nginx-deployment
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=vela-nginx-deployment
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
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
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   vela-nginx-deployment-6c855ccfc8 (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m31s  deployment-controller  Scaled up replica set vela-nginx-deployment-6c855ccfc8 to 4
```

对比上文

```yaml
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
```

我们查看下pod数量是否真的是4个

`kubectl get pods -o wide`

输出

```
NAME                                     READY   STATUS    RESTARTS   AGE   IP               NODE                NOMINATED NODE   READINESS GATES
vela-nginx-deployment-6c855ccfc8-b8fmd   1/1     Running   0          5s    192.168.129.76   kubernetes-node1    <none>           <none>
vela-nginx-deployment-6c855ccfc8-jnrtn   1/1     Running   0          5s    192.168.237.15   kubernetes-master   <none>           <none>
vela-nginx-deployment-6c855ccfc8-pmw6d   1/1     Running   0          25h   192.168.237.13   kubernetes-master   <none>           <none>
vela-nginx-deployment-6c855ccfc8-t6x6v   1/1     Running   0          5s    192.168.129.77   kubernetes-node1    <none>           <none>
```

副本数是4个 `STATUS: Running`,并且 `NODE: kubernetes-node1`, `NODE: kubernetes-master` 在不同节点上拓展我们的pod

让我们测试一下,应用是否正常运行

`curl xxx.xxx.xxx.xxx(master节点IP):31679`

返回

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

恭喜！我们成功拓展4个副本来支撑我们的 High Concurrency

当我们并发量下降我们可以节省资源收缩pod数量

`kubectl scale deployments/vela-nginx-deployment --replicas=1`

让我们查看pod 列表，可以通过 watch 实时查看拓展收缩状态

`watch kubectl get pods -o wide`

输出

```
NAME                                     READY   STATUS    RESTARTS   AGE   IP               NODE                NOMINATED NODE   READINESS GATES
vela-nginx-deployment-6c855ccfc8-pmw6d   1/1     Running   0          25h   192.168.237.13   kubernetes-master   <none>           <none>
```

最终变为1个pod，恭喜！我们成功释放我们的资源
