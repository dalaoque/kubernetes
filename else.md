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

新建Tab查看下载

`watch -n 1 "/sbin/ifconfig ens5 | grep bytes"`

输出

```
RX packets 218499  bytes 316651189 (316.6 MB)
TX packets 119404  bytes 10653036 (10.6 MB)
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
