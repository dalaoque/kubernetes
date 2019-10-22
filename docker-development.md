# 构建Docker镜像和上传到私有仓库

## 一、安装 Docker Desktop for Mac & Windows

Docker Desktop for Mac: [官网下载地址](https://hub.docker.com/editions/community/docker-ce-desktop-mac)

Docker Desktop for Windows: [官网下载地址](https://hub.docker.com/editions/community/docker-ce-desktop-windows)

本文步骤略

## 二、基于 Dockerfile build 镜像

相关docker基础请移步docker

举个Web前端nginx栗子

首先编译打包前端项目`yarn build`

在 build 目录创建 Dockerfile 文件，Dockerfile基础知识请移步[官网](https://docs.docker.com/engine/reference/builder/)

```Dockerfile
FROM nginx:alpine

LABEL maintainer="dalaoque@gmail.com" description="xxx xxx frontend container image."

# dist 是前端编译打包后的文件
ADD dist /home/dist # 前端项目目录dist

# nginx 配置文件
ADD build/nginx.conf /etc/nginx/  
```

build目录下创建nginx.conf

```nginx
user  root;
worker_processes  auto;
pcre_jit on;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    tcp_nopush     	on;

    keepalive_timeout  65;

    gzip  on;
    gzip_static on;

    include /etc/nginx/conf.d/*.conf;

	# DEV
	server {
		listen	9000;
		server_name xxx-dev.xxx.works;
		charset utf-8;

		location ~ /xxx_api {
			proxy_pass				https://xxx-dev.xxx.works;
			proxy_set_header        Upgrade $http_upgrade;
			proxy_set_header        Connection "upgrade";
			proxy_set_header        X-real-ip $remote_addr;
			proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
		}

		location /health {
			add_header content-type 'application/json; charset=utf-8';
			return 200 'online';
		}

		location / {
			root /home/dist;
			index index.html index.htm;
			rewrite ^/.*/$ / last;
			rewrite ^([^.]*[^/])$ $1/ permanent;
		}
	}
}
```

构建 docker image

`docker build -f build/Dockerfile . -t xxx`

输出

```
Sending build context to Docker daemon  7.074MB
Step 1/4 : FROM nginx:alpine
 ---> 4d3c246dfef2
Step 2/4 : LABEL maintainer="dalaoque@gmail.com" description="xxx xxx frontend container image."
 ---> Using cache
 ---> 17552940957e
Step 3/4 : ADD dist /home/dist
 ---> 9a75d5d1d1a0
Step 4/4 : ADD build/nginx.conf /etc/nginx/
 ---> 4c303fdc2d4b
Successfully built 4c303fdc2d4b
Successfully tagged xxx:latest
✨  Done in 1.84s.
```

如果 Sending build context to Docker daemon  7.074MB 文件过大请在项目根目录创建 .dockerignore 文件

Web前端项目 ignore 依赖包和无用的资源

.dockerignore文件内容如下

```

node_modules

.git

```

查看构建后的镜像

`docker images`

输出

```
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
xxx                                            latest              4c303fdc2d4b        5 minutes ago       21.2MB
```

## 三、上传镜像到私有仓库

首先登陆私有仓库,按照提示输入用户名密码，用户名不需要邮箱后缀

`docker login artifactory.xxx.works`

给镜像打tag，同时注明私有仓库地址

docker-pl 为测试镜像仓库

`docker tag xxx artifactory.xxx.works/docker-pl/xxx:1.0`

查看image列表

`docker images`

输出

```
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
artifactory.xxx.works/docker-pl/xxx        1.0                 4c303fdc2d4b        12 minutes ago      21.2MB
xxx                                            latest              4c303fdc2d4b        12 minutes ago      21.2MB
```

推送xxx:1.0到私有仓库

`docker push artifactory.xxx.works/docker-pl/xxx:1.0`

输出

```
The push refers to repository [artifactory.xxx.works/docker-pl/xxx]
b3003ef28992: Pushed
4e6dd69514af: Pushed
e2a556e0495e: Pushed
03901b4a2ea8: Pushed
1.0: digest: sha256:c97a7f15bbb4f4ff9f21cd2d58518f4afe83feaa4fd9078de20afb4dd2d07eb4 size: 1153
```

上传latest版本镜像

步骤和上述相似，只不过不指定版本号，建议每次都push一个latest版本

```bash
docker tag xxx artifactory.xxx.works/docker-pl/xxx && \
docker push artifactory.xxx.works/docker-pl/xxx
```
