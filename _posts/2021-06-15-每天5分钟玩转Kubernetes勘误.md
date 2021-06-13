---
layout: mypost
title: 每天5分钟玩转Kubernetes勘误
categories: [Kubernetes]
---

### Chapter 1：先把Kubernetes跑起来

#### 1.3 部署应用

```shell
kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port 8080
```

* 问题：<font color=red>发现只创建pod，并未创建部署</font>

  

修改为：

```shell
kubectl create deployment kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1
```



### Chapter 4：Kubernetes架构

#### 4.4 用例子把它们串起来

```shell
kubectl run httpd-app --image=httpd --replicas=2
```

* 问题：<font color=red>在K8S v1.18.0以后，–replicas已弃用 ,推荐用 deployment 创建 pods</font>

```shell
Flag --replicas has been deprecated, has no effect and will be removed in the future.
```



修改为：

```shell
kubectl create deployment httpd-app --image=httpd

kubectl scale deployment/httpd-app --replicas=2
```



### Chapter 5：运行应用

#### 5.1.1 运行Deployment

```shell
kubectl run nginx-deployment --image=nginx:1.7.9 --replicas=2
```

* 问题：<font color=red>发现只创建pod，并未创建部署</font>



修改为：

```shell
kubectl create deployment nginx-deployment --image=nginx:1.7.9

kubectl scale deployment/nginx-deployment --replicas=2
```



---

#### 5.1.3 Deployment配置文件简介

通过yaml文件部署应用（nginx-deployment.yaml）：

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: web_server
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
```

* 问题：<font color=red>版本升级导致yaml语法变化</font>

```shell
error: unable to recognize "nginx-deployment.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"
```



修改为：

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

* 注意：spec中必须添加副本标签与Deployment控制器进行匹配<font color=red>（添加selector标签参数，与Pod和deployment相匹配）</font>



---

#### 5.1.5 Failover（没有解决）

在关闭node2后节点状态变为NotReady，与书上一致

![1](1.png)



<font color=red>但node2上跑的pod状态并没有变成Unknown，也并没有在node1上重新创建新的pod</font>，与书上不一致

![2](2.png)



并且在使用kubectl删除原部署并重新启动部署后，<font color=red>node2的pod无法正常删除</font>

![3](3.png)



直到重启node2，才成功删除原pod

![4](4.png)



---

#### 5.2.3 运行自己的DaemonSet

通过yml文件部署DaemonSet（node-exporter.yml）：

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: node-exporter-daemonset
spec:
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      hostNetwork: true
      containers:
      - name: node-exporter
        image: prom/node-exporter
        imagePullPolicy: IfNotPresent
        command:
        - /bin/node_exporter
        - --path.procfs
        - /host/proc
        - --path.sysfs
        - /host/sys
        - --collector.filesystem.ignored-mount-points
        - ^/(sys|proc|dev|host|etc)($|/)
        volumeMounts:
        - name: proc
          mountPath: /host/proc
        - name: sys
          mountPath: /host/sys
        - name: root
          mountPath: /rootfs
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      - name: root
        hostPath:
          path: /
```

* 问题：<font color=red>版本升级导致yaml语法变化</font>

  ```shell
  error: unable to recognize "node_exporter.yml": no matches for kind "DaemonSet" in version "extensions/v1beta1"
  ```



修改yaml文件为：

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter-daemonset
spec:
  selector:
    matchLabels:
      app: node-exporter-daemonset
  template:
    metadata:
      labels:
        app: node-exporter-daemonset
    spec:
      hostNetwork: true
      containers:
      - name: node-exporter
        image: prom/node-exporter
        imagePullPolicy: IfNotPresent
        command:
        - /bin/node_exporter
        - --path.procfs
        - /host/proc
        - --path.sysfs
        - /host/sys
        - --collector.filesystem.ignored-mount-points
        - ^/(sys|proc|dev|host|etc)($|/)
        volumeMounts:
        - name: proc
          mountPath: /host/proc
        - name: sys
          mountPath: /host/sys
        - name: root
          mountPath: /rootfs
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      - name: root
        hostPath:
          path: /
```
