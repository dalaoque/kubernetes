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

安装juju

`sudo snap install juju --classic`

输出

`juju 2.6.9 from Canonical✓ installed`


安装本地云 `localhost` 安装过程中很慢，跟网络有关系

可以查看 juju 相关状态, 请再开启一个终端Tab `sudo watch -c juju status --color`

`sudo juju bootstrap localhost`

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

他大爷终于完成了,没有解决方案，目前只能等

Juju创建一个默认模型，但是为每个项目创建一个新模型很有用

`juju add-model k8s`

输出

`Added 'k8s' model on localhost/localhost with credential 'localhost' for user 'admin'`

部署Charmed Kubernetes了,速度很慢，要有心理准备，目前没有解决方案

`sudo juju deploy charmed-kubernetes`

输出

```
Located bundle "cs:bundle/charmed-kubernetes-270"
Resolving charm: cs:~containers/containerd-33
Resolving charm: cs:~containers/easyrsa-278
Resolving charm: cs:~containers/etcd-460
Resolving charm: cs:~containers/flannel-450
Resolving charm: cs:~containers/kubeapi-load-balancer-682
Resolving charm: cs:~containers/kubernetes-master-754
Resolving charm: cs:~containers/kubernetes-worker-590
Executing changes:
- upload charm cs:~containers/containerd-33 for series bionic
```

## 抓包

下载太慢抓个包，查看网卡

`sudo tcpdump -D`

```
1.lxdbr0 [Up, Running]
2.ens5 [Up, Running]
3.any (Pseudo-device that captures on all interfaces) [Up, Running]
4.lo [Up, Running, Loopback]
5.nflog (Linux netfilter log (NFLOG) interface)
6.nfqueue (Linux netfilter queue (NFQUEUE) interface)
```

抓包

`sudo tcpdump -i ens5 -w dump.pcap -v`

下载到本地计算机

`sudo scp -i "k8s-test-yunhan.pem" -r ubuntu@10.192.35.185:/home/ubuntu/dump.pcap ~/Downloads`

下载 Wireshark 查看工具

## 添加凭证

`sudo juju add-credential aws-china`

请登录 aws 进入 My Security Credentials 页面创建自己的 Access keys，下载csv文件获取 access key 和 secret key,填入命令行


## 自定义部署 Charmed Kubernetes

如果不适用默认套件安装请下载对应套件比如 网络模型插件、kubernetes 版本，请参考[官方文档](https://ubuntu.com/kubernetes/docs/install-manual)

下载 Calico Networking

`sudo curl -o ~/calico-overlay.yaml https://raw.githubusercontent.com/charmed-kubernetes/bundle/master/overlays/calico-overlay.yaml`

下载 AWS Cloud integration

`sudo curl -o ~/aws-overlay.yaml https://raw.githubusercontent.com/charmed-kubernetes/bundle/master/overlays/aws-overlay.yaml`

部署 kubernetes 1.16 + Calico Networking + AWS Cloud integration

过程中可能遇到网络问题比如超时 导致文件下载失败等其他错误，请多试几次或者“科学上网”


`sudo juju deploy cs:~containers/charmed-kubernetes-270 --overlay aws-overlay.yaml --trust --overlay calico-overlay.yaml`

成功后安装kubectl

```bash
sudo mkdir -p ~/.kube && \
sudo juju scp kubernetes-master/0:config ~/.kube/config && \
sudo snap install kubectl --classic && \
kubectl cluster-info
```

