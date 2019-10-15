# 通过 Charmed 安装 Kubernetes

## 在 AWS-China 申请一台 EC2

步骤略

## 安装装 Charmed Kubernetes

参考[官方文档](https://ubuntu.com/kubernetes/docs/install-local)安装 local host Charmed Kubernetes

正常情况下AWS EC2 Ubuntu Server 18.04 默认安装 snap

如果未安装请自行安装

```bash
sudo apt-get update && \
sudo apt-get install snap
```

如果以前尚未安装LXD

`sudo snap install lxd`

查看 lxd 是否安装成功

`lxd version` 输出 `3.0.3`

运行LXD初始化脚本

`sudo lxd init` || `sudo /snap/bin/lxd init`

输出

```bash
Would you like to use LXD clustering? (yes/no) [default=no]:
What name should be used to identify this node in the cluster? [default=ip-10-192-35-55]: k8s-master-yunhan
What IP address or DNS name should be used to reach this node? [default=10.192.35.55]:
Are you joining an existing cluster? (yes/no) [default=no]:
Setup password authentication on the cluster? (yes/no) [default=yes]:
Trust password for new clients: 
Again: 
Do you want to configure a new local storage pool? (yes/no) [default=yes]:
Name of the storage backend to use (btrfs, dir, lvm) [default=btrfs]: dir
Do you want to configure a new remote storage pool? (yes/no) [default=no]:
Would you like to connect to a MAAS server? (yes/no) [default=no]:
Would you like to create a new local network bridge? (yes/no) [default=yes]:
What should the new bridge be called? [default=lxdbr0]:
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]:
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]:none
Would you like LXD to be available over the network? (yes/no) [default=no]: yes
Address to bind LXD to (not including port) [default=all]:
Port to bind LXD to [default=8443]:
Would you like stale cached images to be updated automatically? (yes/no) [default=yes]:
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:
```

大部分按回车默认配置只需如下5项需要填写配置

```
Name of the storage backend to use (btrfs, dir, lvm) [default=btrfs]: dir
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]:none
Would you like LXD to be available over the network? (yes/no) [default=no]: yes
Trust password for new clients: 
Again: 
```

安装juju

`sudo snap install juju --classic`

输出

`juju 2.6.9 from Canonical✓ installed`

查看juju 支持的云

`sudo juju clouds`

输出

```
Since Juju 2 is being run for the first time, downloading latest cloud information.
Fetching latest public cloud list...
This client's list of public clouds is up to date, see `juju clouds --local`.
Cloud           Regions  Default          Type        Description
aws                  19  us-east-1        ec2         Amazon Web Services
aws-china             2  cn-north-1       ec2         Amazon China
aws-gov               2  us-gov-west-1    ec2         Amazon (USA Government)
azure                30  centralus        azure       Microsoft Azure
azure-china           2  chinaeast        azure       Microsoft Azure China
cloudsigma           12  dub              cloudsigma  CloudSigma Cloud
google               18  us-east1         gce         Google Cloud Platform
joyent                6  us-east-1        joyent      Joyent Cloud
oracle                4  us-phoenix-1     oci         Oracle Cloud Infrastructure
oracle-classic        5  uscom-central-1  oracle      Oracle Cloud Infrastructure Classic
rackspace             6  dfw              rackspace   Rackspace Cloud
localhost             1  localhost        lxd         LXD Container Hypervisor
```

添加 localhost credential

`sudo juju add-credential localhost`

输出

```
Enter credential name: admin

Auth Types
  certificate
  interactive

Select auth type [interactive]: interactive

Enter trust-password:

Generating client cert/key in "/root/.local/share/juju/lxd"
Uploaded certificate to LXD server.
Credential "admin" added locally for cloud "localhost".
```

查看 凭证是否添加成功

`sudo juju credentials`

输出

```
Cloud      Credentials
localhost  admin
```

安装本地云 `localhost` 安装过程中很慢，跟网络有关系

可以查看 juju 相关状态, 请再开启一个终端Tab `sudo watch -c juju status --color`

`sudo juju bootstrap --show-log --debug localhost my-controller`

输出

```text
Since Juju 2 is being run for the first time, downloading latest cloud information.
Fetching latest public cloud list...
This client's list of public clouds is up to date, see `juju clouds --local`.
Creating Juju controller "localhost-localhost" on localhost/localhost
Looking for packaged Juju agent version 2.6.9 for amd64
To configure your system to better support LXD containers, please see: https://github.com/lxc/lxd/blob/master/doc/production-setup.md
Launching controller instance(s) on localhost/localhost...
 - juju-9d9d5c-0 (arch=amd64)
Installing Juju agent on bootstrap instance
Fetching Juju GUI 2.15.0
Waiting for address
Attempting to connect to 10.29.141.93:22
Connected to 10.29.141.93
Running machine configuration script...
Bootstrap agent now started
Contacting Juju controller at 10.29.141.93 to verify accessibility...

Bootstrap complete, controller "localhost-localhost" now is available
Controller machines are in the "controller" model
Initial model "default" added
```

他大爷终于完成了,没有解决方案，网络问题

查看 controllers 

`sudo juju controllers`

输出

```
Use --refresh option with this command to see the latest information.

Controller            Model    User   Access     Cloud/Region         Models  Nodes    HA  Version
my-controller*  default  admin  superuser  localhost/localhost       2      1  none  2.6.9
```

如果遇到问题可以删除controller

`sudo juju kill-controller my-controller`

Juju创建一个默认模型，但是为每个项目创建一个新模型很有用

`sudo juju add-model k8s`

输出

`Added 'k8s' model on localhost/localhost with credential 'localhost' for user 'admin'`

新建终端Tab 执行 `sudo juju debug-log -m k8s -n 20` 查看

部署Charmed Kubernetes了,速度很慢，要有心理准备

```shell
sudo curl -o ~/calico-overlay.yaml https://raw.githubusercontent.com/charmed-kubernetes/bundle/master/overlays/calico-overlay.yaml && \

sudo juju deploy cs:~containers/charmed-kubernetes-270 --overlay --overlay calico-overlay.yaml
```

或者安装默认组件

`sudo juju deploy charmed-kubernetes`

输出

```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   303  100   303    0     0   3091      0 --:--:-- --:--:-- --:--:--  3091
Located bundle "cs:~containers/bundle/charmed-kubernetes-270"
Resolving charm: cs:~containers/calico
Resolving charm: cs:~containers/containerd-33
Resolving charm: cs:~containers/easyrsa-278
Resolving charm: cs:~containers/etcd-460
Resolving charm: cs:~containers/kubeapi-load-balancer-682
Resolving charm: cs:~containers/kubernetes-master-754
Resolving charm: cs:~containers/kubernetes-worker-590
Executing changes:
- upload charm cs:~containers/calico-669 for series bionic
- deploy application calico on bionic using cs:~containers/calico-669
  added resource calico
  added resource calico-arm64
  added resource calico-node-image
  added resource calico-upgrade
  added resource calico-upgrade-arm64
- set annotations for calico
- upload charm cs:~containers/containerd-33 for series bionic
- deploy application containerd on bionic using cs:~containers/containerd-33
- set annotations for containerd
- upload charm cs:~containers/easyrsa-278 for series bionic
- deploy application easyrsa on bionic using cs:~containers/easyrsa-278
  added resource easyrsa
- set annotations for easyrsa
- upload charm cs:~containers/etcd-460 for series bionic
- deploy application etcd on bionic using cs:~containers/etcd-460
  added resource core
  added resource etcd
  added resource snapshot
- set annotations for etcd
- upload charm cs:~containers/kubeapi-load-balancer-682 for series bionic
- deploy application kubeapi-load-balancer on bionic using cs:~containers/kubeapi-load-balancer-682
- expose kubeapi-load-balancer
- set annotations for kubeapi-load-balancer
- upload charm cs:~containers/kubernetes-master-754 for series bionic
- deploy application kubernetes-master on bionic using cs:~containers/kubernetes-master-754
  added resource cdk-addons
  added resource core
  added resource kube-apiserver
  added resource kube-controller-manager
  added resource kube-proxy
  added resource kube-scheduler
  added resource kubectl
- set annotations for kubernetes-master
- upload charm cs:~containers/kubernetes-worker-590 for series bionic
- deploy application kubernetes-worker on bionic using cs:~containers/kubernetes-worker-590
  added resource cni-amd64
  added resource cni-arm64
  added resource cni-s390x
  added resource core
  added resource kube-proxy
  added resource kubectl
  added resource kubelet
- expose kubernetes-worker
- set annotations for kubernetes-worker
- add relation kubernetes-master:kube-api-endpoint - kubeapi-load-balancer:apiserver
- add relation kubernetes-master:loadbalancer - kubeapi-load-balancer:loadbalancer
- add relation kubernetes-master:kube-control - kubernetes-worker:kube-control
- add relation kubernetes-master:certificates - easyrsa:client
- add relation etcd:certificates - easyrsa:client
- add relation kubernetes-master:etcd - etcd:db
- add relation kubernetes-worker:certificates - easyrsa:client
- add relation kubernetes-worker:kube-api-endpoint - kubeapi-load-balancer:website
- add relation kubeapi-load-balancer:certificates - easyrsa:client
- add relation containerd:containerd - kubernetes-worker:container-runtime
- add relation containerd:containerd - kubernetes-master:container-runtime
- add relation calico:etcd - etcd:db
- add relation calico:cni - kubernetes-master:cni
- add relation calico:cni - kubernetes-worker:cni
- add unit easyrsa/0 to new machine 0
- add unit etcd/0 to new machine 1
- add unit etcd/1 to new machine 2
- add unit etcd/2 to new machine 3
- add unit kubeapi-load-balancer/0 to new machine 4
- add unit kubernetes-master/0 to new machine 5
- add unit kubernetes-master/1 to new machine 6
- add unit kubernetes-worker/0 to new machine 7
- add unit kubernetes-worker/1 to new machine 8
- add unit kubernetes-worker/2 to new machine 9
Deploy of bundle completed.
```

再开一个终端Tab,查看部署后的状态

`watch juju show-status`

输出

```
Model  Controller           Cloud/Region         Version  SLA          Timestamp
k8s    localhost-localhost  localhost/localhost  2.6.9    unsupported  09:00:16Z

App                    Version  Status  Scale  Charm                  Store       Rev  OS      Notes
calico                          active      5  calico                 jujucharms  669  ubuntu
containerd                      active      5  containerd             jujucharms   33  ubuntu
easyrsa                3.0.1    active      1  easyrsa                jujucharms  278  ubuntu
etcd                   3.2.10   active      3  etcd                   jujucharms  460  ubuntu
kubeapi-load-balancer  1.14.0   active      1  kubeapi-load-balancer  jujucharms  682  ubuntu  exposed
kubernetes-master      1.15.4   active      2  kubernetes-master      jujucharms  754  ubuntu
kubernetes-worker      1.15.4   active      3  kubernetes-worker      jujucharms  590  ubuntu  exposed

Unit                      Workload  Agent  Machine  Public address  Ports           Message
easyrsa/0*                active    idle   0        10.204.182.22                   Certificate Authority connected.
etcd/0*                   active    idle   1        10.204.182.241  2379/tcp        Healthy with 3 known peers
etcd/1                    active    idle   2        10.204.182.64   2379/tcp        Healthy with 3 known peers
etcd/2                    active    idle   3        10.204.182.187  2379/tcp        Healthy with 3 known peers
kubeapi-load-balancer/0*  active    idle   4        10.204.182.70   443/tcp         Loadbalancer ready.
kubernetes-master/0*      active    idle   5        10.204.182.18   6443/tcp        Kubernetes master running.
  calico/3                active    idle            10.204.182.18                   Calico is active
  containerd/3            active    idle            10.204.182.18                   Container runtime available
kubernetes-master/1       active    idle   6        10.204.182.242  6443/tcp        Kubernetes master running.
  calico/4                active    idle            10.204.182.242                  Calico is active
  containerd/4            active    idle            10.204.182.242                  Container runtime available
kubernetes-worker/0       active    idle   7        10.204.182.59   80/tcp,443/tcp  Kubernetes worker running.
  calico/1                active    idle            10.204.182.59                   Calico is active
  containerd/1            active    idle            10.204.182.59                   Container runtime available
kubernetes-worker/1*      active    idle   8        10.204.182.5    80/tcp,443/tcp  Kubernetes worker running.
  calico/0*               active    idle            10.204.182.5                    Calico is active
  containerd/0*           active    idle            10.204.182.5                    Container runtime available
kubernetes-worker/2       active    idle   9        10.204.182.178  80/tcp,443/tcp  Kubernetes worker running.
  calico/2                active    idle            10.204.182.178                  Calico is active
  containerd/2            active    idle            10.204.182.178                  Container runtime available

Machine  State    DNS             Inst id        Series  AZ  Message
0        started  10.204.182.22   juju-3835ef-0  bionic      Running
1        started  10.204.182.241  juju-3835ef-1  bionic      Running
2        started  10.204.182.64   juju-3835ef-2  bionic      Running
3        started  10.204.182.187  juju-3835ef-3  bionic      Running
4        started  10.204.182.70   juju-3835ef-4  bionic      Running
5        started  10.204.182.18   juju-3835ef-5  bionic      Running
6        started  10.204.182.242  juju-3835ef-6  bionic      Running
7        started  10.204.182.59   juju-3835ef-7  bionic      Running
8        started  10.204.182.5    juju-3835ef-8  bionic      Running
9        started  10.204.182.178  juju-3835ef-9  bionic      Running
```

等待status 为 active 或者 started 状态为正常运行时(等待时间30分钟以上)然后导入kubernetes集群配置

下载kubectl

`sudo snap install kubectl --classic`

`sudo juju scp kubernetes-master/0:config ~/.kube/config`

查看kubernetes集群`kubectl get all -o wide`

输出

```
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   19m   <none>
```

查看节点`kubectl get nodes -o wide` 输出

```
NAME            STATUS   ROLES    AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
juju-3835ef-7   Ready    <none>   7m33s   v1.15.4   10.204.182.59    <none>        Ubuntu 18.04.3 LTS   4.15.0-58-generic   containerd://1.2.6
juju-3835ef-8   Ready    <none>   7m32s   v1.15.4   10.204.182.5     <none>        Ubuntu 18.04.3 LTS   4.15.0-58-generic   containerd://1.2.6
juju-3835ef-9   Ready    <none>   7m31s   v1.15.4   10.204.182.178   <none>        Ubuntu 18.04.3 LTS   4.15.0-58-generic   containerd://1.2.6
```

查看kubectl配置 `kubectl cluster-info` 输出

```
Kubernetes master is running at https://10.204.182.70:443
Heapster is running at https://10.204.182.70:443/api/v1/namespaces/kube-system/services/heapster/proxy
CoreDNS is running at https://10.204.182.70:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://10.204.182.70:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
Grafana is running at https://10.204.182.70:443/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
InfluxDB is running at https://10.204.182.70:443/api/v1/namespaces/kube-system/services/monitoring-influxdb:http/proxy
```

在打开一个撞断Tab，让我们来启动一个proxy 暴露 kubernetes-dashboard

`kubectl proxy --address='0.0.0.0'  --accept-hosts='^*$'`

其他地电脑访问 kubernetes-dashboard web 界面

`http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/`

我们的 Charmed Kubernete 就已经完成了
