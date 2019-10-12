# Kubernetes 集群环境安装搭建

建议1. 在公有云申请 `Ubuntu 18.04` 虚机 ，因为基础软件工具比较齐全，否则安装步骤过程中会遇到阻碍，我们尽量把时间精力放在kubernetes 集群相关范围内，争取少走弯路。

建议2. 尽量选择墙外云服务，如果运用本地虚拟机安装 Kubernetes 遇到网络问题需要同学自行“科学上网”，"科学上网"需要自行解决 ^_^，比如更改国内安装源或者自搭梯子等，本文不叙述

建议3. 此文档仅支持 Unbuntu 18.04、Kubernetes 1.16、Calico 3.9，各个组件严格对应版本号，如果版本不对应会出现莫名其妙的Bug，请参考对应版本的官方文档进行安装

## 一、申请一台虚机作为 kubernetes-master
- AMD64处理器 2CPU 一定要2核以上，否则集群初始化会被 block
- 2GB RAM
- 10GB可用磁盘空间
- RedHat Enterprise Linux 7.x +，CentOS 7.x +，Ubuntu 16.04+或Debian 9.x +

### root 权限

``` sudo su ```

### 检查Linux内核版本

```uname -a```

### 查看Ubuntu版本

```cat /etc/lsb-release```

### 更新Ubuntu 从Ubuntu存储库安装Docker

如需 "科学上网" 请自行更改源地址

`kubeadm` 依赖 `docker` 环境安装 Kubernetes

```sudo apt-get update && sudo apt install -y docker.io```

### 启动Docker并使用systemctl命令将其添加到引导时间

`systemctl start docker && systemctl enable docker`

### 查看docker版本信息

`docker --version`

### 安装kubernetes环境

如需 "科学上网" 请自行更改源地址

```
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add && \
sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list && \
sudo apt-get update && \
sudo apt-get install -y kubelet=1.16.0-00 kubeadm=1.16.0-00 kubectl=1.16.0-00 --allow-unauthenticated && \
sudo apt-mark hold kubelet kubeadm kubectl
```

检查软件版本

`apt-cache policy <package-name>`

### 初始化集群 kubernetes-master

> master的CPU核心必须在两个或以上。
> 集群初始化，其中第一个IP是你的内网IP，第二个是你希望Pod所使用的IP地址范围，牢记最后的输出，后面会用

```kubeadm init --apiserver-advertise-address=本机ip --pod-network-cidr=192.168.0.0/16```

or 

```kubeadm init --pod-network-cidr=192.168.0.0/16```

如果 initial 成功 会得到以下信息

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join xxx.xxx.xxx.xxx:6443 --token pihjd2.1w4blf6pftxnpiia \
    --discovery-token-ca-cert-hash sha256:3920ef4600ab1034ff3d7ca2a8aed7b812203014a97790e81a5eaf0c16941ded
```

- 如果失败执行 `kubeadm reset` 后重新 initial

### 执行本地配置

```
mkdir -p $HOME/.kube && \
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && \
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 配置网络模型 安装网络插件Calico
> 本次配置选择 [Calico](https://docs.projectcalico.org/v3.9/getting-started/kubernetes/)

1. Calico (需要init时–pod-network-cidr=192.168.0.0/16)

```
kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
```

2. 确认所有Pod正在运行

```
watch kubectl get pods --all-namespaces
```

等到每个 pods 具有 STATUS 的 Running

```
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-6895d4984b-skqcw    1/1     Running   0          3m11s
kube-system   calico-node-4ftqd                           1/1     Running   0          3m11s
kube-system   coredns-5644d7b6d9-45swj                    1/1     Running   0          4m45s
kube-system   coredns-5644d7b6d9-l2llk                    1/1     Running   0          4m45s
kube-system   etcd-kubernetes-master                      1/1     Running   0          4m3s
kube-system   kube-apiserver-kubernetes-master            1/1     Running   0          4m5s
kube-system   kube-controller-manager-kubernetes-master   1/1     Running   0          4m7s
kube-system   kube-proxy-vj2qm                            1/1     Running   0          4m45s
kube-system   kube-scheduler-kubernetes-master            1/1     Running   0          3m58s
```

3. 按 CTRL + C 退出 `watch`

4. 移除 Taints，避免 pod 被分配到不合适的节点上

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

返回以下内容

```
node/<your-hostname> untainted
```

5. 确认集群中现在有一个节点

```
kubectl get nodes -o wide
```

返回

```
NAME                STATUS   ROLES    AGE   VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kubernetes-master   Ready    master   10m   v1.16.0   xxx.xxx.xxx.xxx   <none>        Ubuntu 18.04.3 LTS   4.15.0-58-generic   docker://18.9.7
```

## 二、申请一台虚机作为 node 节点 kubernetes-node1

### kubernetes-node1 节点加入集群

- 参考上述 安装 docker kubelet kubeadm kubectl
- 执行上述 kubeadm init 成功的结果最后一行命令

```
kubeadm join xxx.xxx.xxx.xxx:6443 --token pihjd2.1w4blf6pftxnpiia \
    --discovery-token-ca-cert-hash sha256:3920ef4600ab1034ff3d7ca2a8aed7b812203014a97790e81a5eaf0c16941ded
```

- 添加节点成功后 在 `kubernetes-master` 节点执行

`watch kubectl get nodes`

- 稍等几秒钟 所有节点 `STATUS` 状态为 `Ready` 说明 简单的 Kubernetes 集群环境完成了

```
NAME                STATUS   ROLES    AGE   VERSION
kubernetes-master   Ready    master   98m   v1.16.0
kubernetes-node1    Ready    <none>   96m   v1.16.0
```

我们查看下 node 节点 详情

`kubectl get node`

输出

```yaml
Name:               kubernetes-master
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=kubernetes-master
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: xxx.xxx.xxx.xxx/20
                    projectcalico.org/IPv4IPIPTunnelAddr: 192.168.237.0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 24 Sep 2019 11:55:35 +0000
Taints:             <none>
Unschedulable:      false
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Thu, 26 Sep 2019 07:22:03 +0000   Thu, 26 Sep 2019 07:22:03 +0000   CalicoIsUp                   Calico is running on this node
  MemoryPressure       False   Fri, 27 Sep 2019 08:44:44 +0000   Tue, 24 Sep 2019 11:55:30 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Fri, 27 Sep 2019 08:44:44 +0000   Tue, 24 Sep 2019 11:55:30 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Fri, 27 Sep 2019 08:44:44 +0000   Tue, 24 Sep 2019 11:55:30 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Fri, 27 Sep 2019 08:44:44 +0000   Tue, 24 Sep 2019 11:57:46 +0000   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  xxx.xxx.xxx.xxx
  Hostname:    kubernetes-master
Capacity:
 cpu:                2
 ephemeral-storage:  60795672Ki
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             2041272Ki
 pods:               110
Allocatable:
 cpu:                2
 ephemeral-storage:  56029291223
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             1938872Ki
 pods:               110
System Info:
 Machine ID:                 d502558aca414f9f882a4ad5cb76c326
 System UUID:                D502558A-CA41-4F9F-882A-4AD5CB76C326
 Boot ID:                    c6b72771-bcf8-42f5-aa75-201e23c63066
 Kernel Version:             4.15.0-64-generic
 OS Image:                   Ubuntu 18.04.3 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://18.9.7
 Kubelet Version:            v1.16.0
 Kube-Proxy Version:         v1.16.0
PodCIDR:                     192.168.0.0/24
PodCIDRs:                    192.168.0.0/24
Non-terminated Pods:         (10 in total)
  Namespace                  Name                                         CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                                         ------------  ----------  ---------------  -------------  ---
  default                    vela-nginx-deployment-6c855ccfc8-pmw6d       0 (0%)        0 (0%)      0 (0%)           0 (0%)         24h
  kube-system                calico-kube-controllers-6895d4984b-skqcw     0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d20h
  kube-system                calico-node-4ftqd                            250m (12%)    0 (0%)      0 (0%)           0 (0%)         2d20h
  kube-system                coredns-5644d7b6d9-45swj                     100m (5%)     0 (0%)      70Mi (3%)        170Mi (8%)     2d20h
  kube-system                coredns-5644d7b6d9-l2llk                     100m (5%)     0 (0%)      70Mi (3%)        170Mi (8%)     2d20h
  kube-system                etcd-kubernetes-master                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d20h
  kube-system                kube-apiserver-kubernetes-master             250m (12%)    0 (0%)      0 (0%)           0 (0%)         2d20h
  kube-system                kube-controller-manager-kubernetes-master    200m (10%)    0 (0%)      0 (0%)           0 (0%)         2d20h
  kube-system                kube-proxy-vj2qm                             0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d20h
  kube-system                kube-scheduler-kubernetes-master             100m (5%)     0 (0%)      0 (0%)           0 (0%)         2d20h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                1 (50%)     0 (0%)
  memory             140Mi (7%)  340Mi (17%)
  ephemeral-storage  0 (0%)      0 (0%)
Events:              <none>


Name:               kubernetes-node1
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=kubernetes-node1
                    kubernetes.io/os=linux
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: xxx.xxx.xxx.xxx/20
                    projectcalico.org/IPv4IPIPTunnelAddr: 192.168.129.64
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 24 Sep 2019 12:18:26 +0000
Taints:             <none>
Unschedulable:      false
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Wed, 25 Sep 2019 11:46:08 +0000   Wed, 25 Sep 2019 11:46:08 +0000   CalicoIsUp                   Calico is running on this node
  MemoryPressure       False   Fri, 27 Sep 2019 08:44:58 +0000   Wed, 25 Sep 2019 11:46:02 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Fri, 27 Sep 2019 08:44:58 +0000   Wed, 25 Sep 2019 11:46:02 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Fri, 27 Sep 2019 08:44:58 +0000   Wed, 25 Sep 2019 11:46:02 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Fri, 27 Sep 2019 08:44:58 +0000   Wed, 25 Sep 2019 11:46:12 +0000   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  xxx.xxx.xxx.xxx
  Hostname:    kubernetes-node1
Capacity:
 cpu:                1
 ephemeral-storage:  25226960Ki
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             1009264Ki
 pods:               110
Allocatable:
 cpu:                1
 ephemeral-storage:  23249166298
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             906864Ki
 pods:               110
System Info:
 Machine ID:                 cd6032d1aa554cefbd617cf6168a78b8
 System UUID:                CD6032D1-AA55-4CEF-BD61-7CF6168A78B8
 Boot ID:                    886b3b64-7f47-4dbe-8d12-a9fe69325930
 Kernel Version:             4.15.0-64-generic
 OS Image:                   Ubuntu 18.04.3 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://18.9.7
 Kubelet Version:            v1.16.0
 Kube-Proxy Version:         v1.16.0
PodCIDR:                     192.168.1.0/24
PodCIDRs:                    192.168.1.0/24
Non-terminated Pods:         (2 in total)
  Namespace                  Name                 CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                 ------------  ----------  ---------------  -------------  ---
  kube-system                calico-node-t4btm    250m (25%)    0 (0%)      0 (0%)           0 (0%)         2d20h
  kube-system                kube-proxy-mdf6r     0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d20h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                250m (25%)  0 (0%)
  memory             0 (0%)      0 (0%)
  ephemeral-storage  0 (0%)      0 (0%)
Events:              <none>
```

恭喜！简单的两个节点的 Kubernetes 集群完成了
