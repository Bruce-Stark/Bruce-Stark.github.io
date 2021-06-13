---
layout: mypost
title: 每天5分钟玩转Kubernetes实战
categories: [Kubernetes]
---

### Chapter 1：先把Kubernetes跑起来

#### 1.3 部署应用

```shell
kubectl create deployment kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1
```



---

#### 1.4 访问应用

```shell
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
```



---

#### 1.5 Scale应用

```shell
kubectl scale deployments/kubernetes-bootcamp --replicas=3
```



---

#### 1.6 滚动更新

```shell
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
```



### Chapter 4：Kubernetes架构

#### 4.4 用例子把它们串起来

```shell
kubectl create deployment httpd-app --image=httpd

kubectl scale deployment/httpd-app --replicas=2
```



### Chapter 5：运行应用

#### 5.1.1 运行Deployment

```shell
kubectl create deployment nginx-deployment --image=nginx:1.7.9

kubectl scale deployment/nginx-deployment --replicas=2
```



---

#### 5.1.3 Deployment配置文件简介

```yaml
apiVersion: apps/v1                 #Api接口版本
kind: Deployment                    #定义控制器
metadata:
  name: nginx-deployment            #deployment名称
spec:
  replicas: 2                       #副本数
  selector:                         #选择标签
    matchLabels:                    #定义标签
      app: nginx-deployment         #标签名 
  template:                         #pod容器
    metadata:
      labels:                       #定义标签
        app: nginx-deployment       #标签名
    spec:
      containers:
      - name: nginx                 #容器名
        image: nginx:1.7.9          #容器镜像
```



---

