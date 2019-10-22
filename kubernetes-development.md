## 启动 Kubernetes

具体教程请谷歌/百度不具体叙述

[推荐一篇教程](https://rominirani.com/tutorial-getting-started-with-kubernetes-with-docker-on-mac-7f58467203fd)

## 创建私有仓库Secret

使用命令行创建

```bash
kubectl create secret docker-registry secret-name \
    --docker-server=artifactory.xxx.works  \
    --docker-username=dalaoque \
    --docker-password=xxxx \
    --docker-email=dalaoque@gmail.com \
    -n default
```

命令解释

```markdown
secret-name:          指定密钥的键名称, 可自行定义
--docker-server:    私有仓库服务地址
--docker-username:  私有仓库用户名
--docker-password:  私有仓库密码
--docker-email:     邮件地址
-n default:                namespace用于隔离环境，不同命名空间使用不同的 secret，默认是 default
```

查看secret列表

`kubectl get secret`

输出

```
NAME                  TYPE                                  DATA   AGE
default-token-2lwx2   kubernetes.io/service-account-token   3      29d
secret-name           kubernetes.io/dockerconfigjson        1      7h46m
```
