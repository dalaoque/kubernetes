# 通过yaml文件部署Services

## NodePort

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-test
spec:
  selector:
    matchLabels:
      app: nginx-test # 声明 pod 的 labels
  replicas: 2 # 拓展pod副本数量
  template:
    metadata:
      labels:
        app: nginx-test # 声明 pod 的 labels 必须和 spec.selector.matchLabels.app 对应
    spec:
      containers:
      - name: nginx
        image: nginx:1.15
        # image: gcr.io/google-samples/kubernetes-bootcamp:v1
        ports:
        - containerPort: 80 # docker 镜像内部服务暴露的端口号，比如 nginx 默认是80

---

apiVersion: v1
kind: Service
metadata:
  name: nginx-test
  labels:
    name: nginx-test
spec:
  type: NodePort      # 这里代表是NodePort类型的
  ports:
  - port: 80          # 这里的端口和clusterIP对应，即 clusterIP:80,供内部访问。
    targetPort: 80  # 端口一定要和container暴露出来的端口对应，nginx 默认是80
    protocol: TCP
    nodePort: 30001   # 所有的节点都会开放此端口，此端口供外部调用。
  selector:
    app: nginx-test          # 这里选择器一定要选择容器的标签即deployment 的 selector。

---

apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-test
spec:
  rules:
  - host: nginx.test.com
    http:
      paths:
      - backend:
          serviceName: nginx-test
          servicePort: 80
```

## Loadbalancer 暴露服务

...
