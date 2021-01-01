### 构建容器镜像
* $ docker build -t kubia .  # Docker 会在目录中寻找Dockerfile 然后基于其中的指令构建镜像
* 构建镜像时，Dockerfile中每一条单独的指令都会创建一个新层

```
# 列出本地存储的镜像
$ docker images 
kubia                                                               latest               d7c5dd569a69   About a minute ago   660MB

# 运行容器镜像 
$ docker run --name kubia-container -p 8080:8080 -d kubia

curl localhost:8080

# 列出所有运行中的容器 
$ docker ps 
➜ docker ps
CONTAINER ID   IMAGE     COMMAND         CREATED         STATUS         PORTS                    NAMES
f580f6ee2582   kubia     "node app.js"   4 minutes ago   Up 4 minutes   0.0.0.0:8080->8080/tcp   kubia-container

# 获取更多的容器信息
$ docker inspect kubia-container

# 在已有容器内部运行shell
$ docker exec -it kubia-container bash

# 从容器内列出进程
root@f580f6ee2582:/# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  1.2 879140 25900 ?        Ssl  01:40   0:00 node app.js
root        13  0.0  0.1  20248  3232 pts/0    Ss   01:48   0:00 bash
root        20  0.0  0.0  17504  2032 pts/0    R+   01:49   0:00 ps aux
root@f580f6ee2582:/#

# 容器拥有完整的文件系统
root@f580f6ee2582:/# ls /
app.js  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# 停止容器 
➜ docker stop kubia-container
kubia-container

# 删除容器
➜ docker rm kubia-container
kubia-container

# 容器附加标注
➜ docker tag kubia chyiyaqing/kubia

# 重新命名镜像
➜ docker images | grep kubia
chyiyaqing/kubia                                                    latest                    d7c5dd569a69   14 hours ago        660MB
kubia                                                               latest                    d7c5dd569a69   14 hours ago        660MB

# Docker Hub 推送镜像 
➜ docker push chyiyaqing/kubia
Using default tag: latest
The push refers to repository [docker.io/chyiyaqing/kubia]
9f3144d7f0e6: Pushed
ab90d83fa34a: Mounted from library/node
8ee318e54723: Mounted from library/node
e6695624484e: Mounted from library/node
da59b99bbd3b: Mounted from library/node
5616a6292c16: Mounted from library/node
f3ed6cb59ab0: Mounted from library/node
654f45ecb7e3: Mounted from library/node
2c40c66f7667: Mounted from library/node
latest: digest: sha256:b01781c2c9dedc4104b4c8a9c251532144c189b8c113591d47aaa14457183b31 size: 2213

# 运行镜像 
➜ docker run -p 8080:8080 -d chyiyaqing/kubia
```

### Minikube Kubernetes集群
```
# 启动minikube 虚拟机
$ minikube start

# 展示集群信息
➜ kubectl cluster-info
Kubernetes master is running at https://192.168.49.2:8443
KubeDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

每个节点运行[Docker, Kubelet, kube-proxy] 可以通过kubectl命令客户端向运行在主节点的Kubernetes API 服务器发出REST请求以及集群交互

# kubectl 列出集群节点
➜ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   19d   v1.19.4

# 查看节点信息
$ kubectl describe node 

# 创建一个容器 
$ kubectl run kubia --image=chyiyaqing/kubia --port=8080

# 一个pod是一组相关的容器 总一起运行在同一个工作节点上,以及同一个Linux命名空间中

# 列出pod
➜ kubectl get pods
NAME    READY   STATUS             RESTARTS   AGE
kubia   0/1     ImagePullBackOff   0          21m

要让pod能从外部访问，需要通过服务对象公开，创建一个LoadBalancer类型的服务，可以通过负载均衡的公共IP访问pod

# 创建一个服务对象
$ kubectl expose po kubia --type=LoadBalancer --name kubia-http 

# 列出服务
➜ kubectl get services
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP          20d
kubia-http   LoadBalancer   10.102.204.234   <pending>     8080:31935/TCP   98s

# 使用外部IP访问服务
➜ minikube service kubia-http
|-----------|------------|-------------|---------------------------|
| NAMESPACE |    NAME    | TARGET PORT |            URL            |
|-----------|------------|-------------|---------------------------|
| default   | kubia-http |        8080 | http://192.168.49.2:31935 |
|-----------|------------|-------------|---------------------------|

服务表示一组或多组提供相同服务的pod的静态地址，到达服务IP和端口的请求将被转发到属于该服务的一个容器的IP和端口

➜ kubectl get pods -o wide
NAME     READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
kubia    1/1     Running   0          32h   172.17.0.3   minikube   <none>           <none>
kubia2   1/1     Running   0          32h   172.17.0.4   minikube   <none>           <none>

➜ kubectl describe pod kubia
Name:         kubia
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Wed, 16 Dec 2020 23:51:57 +0800
Labels:       run=kubia
Annotations:  <none>
Status:       Running
IP:           172.17.0.3
IPs:
  IP:  172.17.0.3
Containers:
  kubia:
    Container ID:   docker://1836daf8933d3bfb8ccf4d1507f5a8d948908c4a827b05b511449b58131d7a15
    Image:          chyiyaqing/kubia
    Image ID:       docker-pullable://chyiyaqing/kubia@sha256:b01781c2c9dedc4104b4c8a9c251532144c189b8c113591d47aaa14457183b31
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 17 Dec 2020 02:51:32 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-fgzpx (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-fgzpx:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-fgzpx
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:          <none>

# 访问Minikube dashboard 
$ minikube dashboard
