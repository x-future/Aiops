k8s实战一

1、设置kubectl输入命令自动补全

```
yum install -y bash-completion
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

2、使用kubectl进行增、删、查、改等常用操作

```
[root@k8s-master01 ~]# kubectl  -h
kubectl controls the Kubernetes cluster manager. 
Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/
Basic Commands (Beginner):    #基本命令集，适合新手
  create         Create a resource from a file or from stdin.
  expose         使用 replication controller, service, deployment 或者 pod 并暴露它作为一个 新的
Kubernetes Service
  run            在集群中运行一个指定的镜像
  set            为 objects 设置一个指定的特征
  run-container  在集群中运行一个指定的镜像. This command is deprecated, use "run" instead

Basic Commands (Intermediate):  #基本命令集，适合有一定基础的人
  get            显示一个或更多 resources
  explain        查看资源的文档
  edit           在服务器上编辑一个资源
  delete         Delete resources by filenames, stdin, resources and names, or by resources and label selector

Deploy Commands:   #发布相关的命令集
  rollout        Manage the rollout of a resource
  rolling-update 完成指定的 ReplicationController 的滚动升级
  scale          为 Deployment, ReplicaSet, Replication Controller 或者 Job 设置一个新的副本数量
  autoscale      自动调整一个 Deployment, ReplicaSet, 或者 ReplicationController 的副本数量

Cluster Management Commands: #集群管理相关的命令集
  certificate    修改 certificate 资源.
  cluster-info   显示集群信息
  top            Display Resource (CPU/Memory/Storage) usage.
  cordon         标记 node 为 unschedulable
  uncordon       标记 node 为 schedulable
  drain          Drain node in preparation for maintenance
  taint          更新一个或者多个 node 上的 taints

Troubleshooting and Debugging Commands:  #故障检测及调试相关命令集
  describe       显示一个指定 resource 或者 group 的 resources 详情
  logs           输出容器在 pod 中的日志
  attach         Attach 到一个运行中的 container
  exec           在一个 container 中执行一个命令
  port-forward   Forward one or more local ports to a pod
  proxy          运行一个 proxy 到 Kubernetes API server
  cp             复制 files 和 directories 到 containers 和从容器中复制 files 和 directories.
  auth           Inspect authorization

Advanced Commands: #高级命令集
  apply          通过文件名或标准输入流(stdin)对资源进行配置
  patch          使用 strategic merge patch 更新一个资源的 field(s)
  replace        通过 filename 或者 stdin替换一个资源
  convert        在不同的 API versions 转换配置文件

Settings Commands:  #设置相关的命令集
  label          更新在这个资源上的 labels
  annotate       更新一个资源的注解
  completion     Output shell completion code for the specified shell (bash or zsh)

Other Commands: #其他命令集
  api-versions   Print the supported API versions on the server, in the form of "group/version"
  config         修改 kubeconfig 文件
  help           Help about any command
  plugin         Runs a command-line plugin
  version        输出 client 和 server 的版本信息

Usage:    #使用格式
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.  #各个子命令如何获取命令帮助
Use "kubectl options" for a list of global command-line options (applies to all commands).  #查看命令的通用选项(所有命令)
```

3、创建一个应用程序

**1)创建一个应用程序，我们使用  "kubectl run " 命令**

```
kubectl run nginx-deploy --image=nginx:1.14-alpine --port=80 --replicas=1 --dry-run=true # #nginx-deploy表示deployment的名称 --image表示镜像的地址 --port表示pod暴露的端口 --replicas表示副本的个数 --dry-run表示测试，不真正执行命令
新版本用create 替代了
[root@k8s-master01 ~]# kubectl run nginx-deploy --image=nginx:1.14-alpine --port=80 --replicas=1  #测试发现命令正常，执行命令
root@k8s-master01 ~]# kubectl get deployment  #我们查看一下deployment的信息，是否有当前创建的
NAME(名称)           DESIRED(需要pod的个数)   CURRENT(当前已经存在的个数)   UP-TO-DATE(最新创建的pod个数)   AVAILABLE(可用的pod个数)   AGE(deployment存活的时间)
nginx-deploy 　　　　　　　　  1        　　　　　　　　 1                     1                              1                        12s
[root@k8s-master01 ~]# kubectl get pod -o wide  #获取pod的信息，-o wide 表示更详细的显示信息
NAME(pod的名称)                READY(就绪的个数/总的个数)     STATUS(目前的状态)    RESTARTS(重启的次数)   AGE(存活的时间)       IP(pod的IP地址)           NODE(部署在哪个节点)
nginx-deploy-7db697dfbd-qkdqp        1/1                       Running              0                   19s            10.244.2.2                  k8s-node02

```

集群内部访问

```
[root@k8s-master01 ~]# curl -I 10.244.2.2  #在集群内进行访问，返回状态码为200，访问没有问题
```

集群外部访问

![img](https://img2018.cnblogs.com/blog/911490/201810/911490-20181030112211059-1463706380.png)



当我们在集群之外访问是发现无法访问，那么集群之外的客户端如何才能访问呢？这就需要我们的service服务了，下面我们就创建一个service，是外部客户端可以访问我们的pod

### 3)创建一个service

```
[root@k8s-master01 ~]# kubectl expose deployment nginx-deploy  --name=nginx   --port=80 --target-port=80 --type=NodePort
```

使用集群外客户端再一次访问，需要使用集群任意节点的IP地址加上暴露的端口号

### 4)现在我们来删除刚刚参加的pod，看看会发生什么

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[root@k8s-master01 ~]# kubectl get pod 
NAME                            READY     STATUS    RESTARTS   AGE
nginx-deploy-7db697dfbd-qkdqp   1/1       Running   0          2h
[root@k8s-master01 ~]# kubectl delete pod nginx-deploy-7db697dfbd-qkdqp #删除deployment下的pod
pod "nginx-deploy-7db697dfbd-qkdqp" deleted
[root@k8s-master01 ~]# kubectl get pod -w  #然后迅速的查看pod状态，-w是一直等待的意思，我们可以看到pod被删除后系统又自动创建一个新的pod  (deployment管理的pod会尽量一直保持我们期望的状态)
NAME                            READY     STATUS              RESTARTS   AGE
nginx-deploy-7db697dfbd-46x7s   0/1       ContainerCreating   0          6s
nginx-deploy-7db697dfbd-46x7s   1/1       Running   0         15s
```

**查看下deployment和service的状态**

```
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1         1         1            1           2h
[root@k8s-master01 ~]# kubectl get svc  #service的名称和IP没有发生改变
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP        2h
nginx        NodePort    10.107.16.3   <none>        80:30757/TCP   1h
```

![img](https://img2018.cnblogs.com/blog/911490/201810/911490-20181030131005219-1414862384.png)

没有问题，为什么我们删除pod之后重新创建service还可创建成功呢？这是因为service和pod直接是使用标签来进行关联的

```
root@k8s-master01 ~]# kubectl describe svc nginx   #查看nginx的信息
Name:                     nginx
Namespace:                default
Labels:                   run=nginx-deploy   #我们看到nginx的标签为run=nginx-deploy
Annotations:              <none>
Selector:                 run=nginx-deploy
Type:                     NodePort
IP:                       10.107.16.3
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30757/TCP
Endpoints:                10.244.1.2:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
[root@k8s-master01 ~]# kubectl describe pod nginx-deploy-7db697dfbd-46x7s #在看下新创建的pod信息
Name:           nginx-deploy-7db697dfbd-46x7s
Namespace:      default
Node:           k8s-node01/172.16.150.213
Start Time:     Tue, 30 Oct 2018 13:06:02 +0800
Labels:         pod-template-hash=3862538968
                run=nginx-deploy  #同样也拥有run=nginx-deploy的标签
Annotations:    <none>
Status:         Running
IP:             10.244.1.2
```

### 5)下面我们来尝试对nginx-deploy这个deployment进行扩容和缩减操作

```
[root@k8s-master01 ~]# kubectl get deployment 
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1         1         1            1           2h
[root@k8s-master01 ~]# kubectl scale --replicas=5 deployment nginx-deploy  #对名称为nginx-deploy类型为deployment的对象进行扩容，有初始的1个扩容到五个
deployment.extensions "nginx-deploy" scaled
[root@k8s-master01 ~]# kubectl get pod -w #我们可以看到pod扩容的过程
NAME                            READY     STATUS              RESTARTS   AGE
nginx-deploy-7db697dfbd-46x7s   1/1       Running             0          14m
nginx-deploy-7db697dfbd-lr7wx   0/1       ContainerCreating   0          1s
nginx-deploy-7db697dfbd-sk48l   0/1       ContainerCreating   0          1s
nginx-deploy-7db697dfbd-tdtc8   0/1       ContainerCreating   0          1s
nginx-deploy-7db697dfbd-xg25w   0/1       ContainerCreating   0          1s
nginx-deploy-7db697dfbd-xg25w   1/1       Running   0         11s
nginx-deploy-7db697dfbd-lr7wx   1/1       Running   0         12s
nginx-deploy-7db697dfbd-sk48l   1/1       Running   0         12s
nginx-deploy-7db697dfbd-tdtc8   1/1       Running   0         12s
```



6.现在我们对nginx-deploy进行滚动升级及回滚操作，由[1.14-alpine](https://github.com/nginxinc/docker-nginx/blob/d377983a14b214fcae4b8e34357761282aca788f/stable/alpine/Dockerfile)` 升级到1.15-alpine，并由1.15-alpine回滚到1.14-alpine版本(nginx在docker hub上版本信息：https://hub.docker.com/_/nginx/)`

**滚动升级：**

```
root@k8s-master01 ~]# kubectl set image deployment nginx-deploy nginx-deploy=nginx:1.15-alpine --record #具体命令及参数含义请参考命令帮助 --record 添加注解Annotations: 
deployment.apps "nginx-deploy" image updated
[root@k8s-master01 ~]# kubectl get pod -w #观察滚动升级的过程
NAME                            READY     STATUS              RESTARTS   AGE
nginx-deploy-6c7dd4d9bf-c58mr   0/1       ContainerCreating   0          3s
nginx-deploy-6c7dd4d9bf-fvmt5   0/1       ContainerCreating   0          3s
nginx-deploy-7db697dfbd-46x7s   1/1       Running             0          27m
nginx-deploy-7db697dfbd-xg25w   1/1       Running             0          13m
nginx-deploy-6c7dd4d9bf-c58mr   1/1       Running   0         28s
nginx-deploy-7db697dfbd-xg25w   1/1       Terminating   0         13m
nginx-deploy-6c7dd4d9bf-lp7w2   0/1       Pending   0         0s
nginx-deploy-6c7dd4d9bf-lp7w2   0/1       Pending   0         0s
nginx-deploy-6c7dd4d9bf-lp7w2   0/1       ContainerCreating   0         0s
nginx-deploy-7db697dfbd-xg25w   0/1       Terminating   0         13m
nginx-deploy-7db697dfbd-xg25w   0/1       Terminating   0         13m
nginx-deploy-7db697dfbd-xg25w   0/1       Terminating   0         13m
nginx-deploy-6c7dd4d9bf-fvmt5   1/1       Running   0         30s
nginx-deploy-7db697dfbd-46x7s   1/1       Terminating   0         27m
nginx-deploy-7db697dfbd-46x7s   0/1       Terminating   0         27m
nginx-deploy-7db697dfbd-46x7s   0/1       Terminating   0         27m
nginx-deploy-7db697dfbd-46x7s   0/1       Terminating   0         27m
^C[root@k8s-master01 ~]# kubectl get pod 
NAME                            READY     STATUS              RESTARTS   AGE
nginx-deploy-6c7dd4d9bf-c58mr   1/1       Running             0          41s
nginx-deploy-6c7dd4d9bf-fvmt5   1/1       Running             0          41s
nginx-deploy-6c7dd4d9bf-lp7w2   0/1       ContainerCreating   0          13s
[root@k8s-master01 ~]# kubectl get pod #查看最后的状态，比对一下现在的pod和之前的pod是否相同
NAME                            READY     STATUS    RESTARTS   AGE
nginx-deploy-6c7dd4d9bf-c58mr   1/1       Running   0          1m
nginx-deploy-6c7dd4d9bf-fvmt5   1/1       Running   0          1m
nginx-deploy-6c7dd4d9bf-lp7w2   1/1       Running   0          45s
```

查看任意一个pod的信息，看看镜像是否升级

```
[root@k8s-master01 ~]# kubectl describe pod nginx-deploy-6c7dd4d9bf-c58mr
Name:           nginx-deploy-6c7dd4d9bf-c58mr
Namespace:      default
Node:           k8s-node02/172.16.150.214
Start Time:     Tue, 30 Oct 2018 13:33:15 +0800
Labels:         pod-template-hash=2738808569
                run=nginx-deploy
Annotations:    <none>
Status:         Running
IP:             10.244.2.5
Controlled By:  ReplicaSet/nginx-deploy-6c7dd4d9bf
Containers:
  nginx-deploy:
    Container ID:   docker://934e8074c90e0a5114ae846a2405515885efbcf1fcba8653a66d303f94e47253
    Image:          nginx:1.15-alpine   #image信息
    Image ID:       docker-pullable://docker.io/nginx@sha256:ae5da813f8ad7fa785d7668f0b018ecc8c3a87331527a61d83b3b5e816a0f03c
......（以下省略）
```

**版本回滚：**

```
[root@k8s-master01 ~]# kubectl rollout undo deployment nginx-deploy     #--to-revision 参数可以指定回退的版本
deployment.apps "nginx-deploy" 
[root@k8s-master01 ~]# kubectl get pod -w
NAME                            READY     STATUS              RESTARTS   AGE
nginx-deploy-6c7dd4d9bf-c58mr   1/1       Running             0          7m
nginx-deploy-6c7dd4d9bf-fvmt5   1/1       Running             0          7m
nginx-deploy-6c7dd4d9bf-lp7w2   0/1       Terminating         0          7m
nginx-deploy-7db697dfbd-gskcv   0/1       ContainerCreating   0          4s
nginx-deploy-7db697dfbd-ssws8   0/1       ContainerCreating   0          4s
nginx-deploy-7db697dfbd-ssws8   1/1       Running   0         11s
nginx-deploy-6c7dd4d9bf-fvmt5   1/1       Terminating   0         7m
nginx-deploy-7db697dfbd-2qh7v   0/1       Pending   0         0s
nginx-deploy-7db697dfbd-2qh7v   0/1       Pending   0         0s
nginx-deploy-7db697dfbd-2qh7v   0/1       ContainerCreating   0         0s
nginx-deploy-6c7dd4d9bf-fvmt5   0/1       Terminating   0         7m
nginx-deploy-6c7dd4d9bf-lp7w2   0/1       Terminating   0         7m
nginx-deploy-6c7dd4d9bf-lp7w2   0/1       Terminating   0         7m
```

  **查看任意一个pod的信息，看看镜像是否回滚到**[1.14-alpine](https://github.com/nginxinc/docker-nginx/blob/d377983a14b214fcae4b8e34357761282aca788f/stable/alpine/Dockerfile)**版本**



```
[root@k8s-master01 ~]# kubectl describe pod nginx-deploy-7db697dfbd-2qh7v  #查看任意一个pod的版本信息，查看是否回滚到1.14版本
Name:           nginx-deploy-7db697dfbd-2qh7v
Namespace:      default
Node:           k8s-node02/172.16.150.214
Start Time:     Tue, 30 Oct 2018 13:40:55 +0800
Labels:         pod-template-hash=3862538968
                run=nginx-deploy
Annotations:    <none>
Status:         Running
IP:             10.244.2.7
Controlled By:  ReplicaSet/nginx-deploy-7db697dfbd
Containers:
  nginx-deploy:
    Container ID:   docker://b75740e5919bd975755b256c83e03b63ea95cf2307ffc606abd03b59fea6634a
    Image:          nginx:1.14-alpine
    Image ID:       docker-pullable://docker.io/nginx@sha256:8976218be775f4244df2a60a169d44606b6978bac4375192074cefc0c7824ddf
```

**下面我们对刚刚操作的命令做一个大致的总结：**

 

```
kubectl run       #创建一个deployment或job来管理创建的容器
kubectl get       #显示一个或多个资源，可以使用标签过滤，默认查看当前名称空间的资源
kubectl expose    #将一个资源暴露为一个新的kubernetes的service资源，资源包括pod (po)， service (svc)， replicationcontroller (rc)，deployment(deploy)， replicaset (rs)
kubectl describe  #显示特定资源或资源组的详细信息
kubectl scale     #可以对Deployment, ReplicaSet, Replication Controller, 或者StatefulSet设置新的值，可以指定一个或多个先决条件
kubectl set       #更改现有的应用程序资源
kubectl rollout   #资源回滚管理
```



[k8s学习笔记之四:资源清单定义入门](https://www.cnblogs.com/panwenbin-logs/p/9881615.html)

### k8s中的资源

#### **1.什么叫资源？**

```
k8s中所有的内容都抽象为资源， 资源实例化之后，叫做对象
```

#### 2.在k8s中有哪些资源？

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
工作负载型资源(workload)： Pod ReplicaSet Deployment StatefulSet DaemonSet Job CronJob (ReplicationController在v1.11版本被废弃)
服务发现及负载均衡型资源(ServiceDiscovery LoadBalance):  Service  Ingress, ...
配置与存储型资源： Volume(存储卷) CSI(容器存储接口,可以扩展各种各样的第三方存储卷)
特殊类型的存储卷：ConfigMap(当配置中心来使用的资源类型)Secret(保存敏感数据) DownwardAPI(把外部环境中的信息输出给容器)
以上这些资源都是配置在名称空间级别 

集群级资源：Namespace Node Role ClusterRole RoleBinding(角色绑定) ClusterRoleBinding(集群角色绑定) 
元数据型资源：HPA(Pod水平扩展) PodTemplate(Pod模板,用于让控制器创建Pod时使用的模板) LimitRange(用来定义硬件资源限制的)
```

### 资源清单

#### 1.什么是资源清单

```
在k8s中，一般使用yaml格式的文件来创建符合我们预期期望的pod，这样的yaml文件我们一般称为资源清单
```

#### 2.资源清单的格式

```
apiVersion: group/apiversion  # 如果没有给定group名称，那么默认为croe，可以使用kubectl api-versions 获取当前k8s版本上所有的apiVersion版本信息(每个版本可能不同)
kind:       #资源类别
metadata：  #资源元数据
   name
   namespace  #k8s自身的namespace
   lables
   annotations   #主要目的是方便用户阅读查找
spec:期望的状态（disired state）
status：当前状态，本字段有kubernetes自身维护，用户不能去定义
```

**#配置清单主要有五个一级字段，其中status用户不能定义，有k8s自身维护**

#### **3.获取资源的apiVersion版本及资源配置的帮助**

**1)获取apiVersion版本信息**

```
[root@k8s-master01 ~]# kubectl api-versions 
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1
apiregistration.k8s.io/v1beta1
apps/v1
apps/v1beta1
apps/v1beta2
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
batch/v1
```

**2)获取资源的apiVersion版本信息**

```
root@k8s-master01 ~]# kubectl explain pod 
KIND:     Pod
VERSION:  v1 #注意doc里面的版本信息可能会旧
.....(以下省略)
[root@k8s-master01 ~]# kubectl explain Ingress
KIND:     Ingress
VERSION:  extensions/v1beta1
```

#### 3)获取资源配置清单中字段设置帮助文档(以pod为例)

**获取pod资源的配置清单一级字段**

```
[root@k8s-master01 ~]# kubectl explain pod
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion    <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#resources

   kind    <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds
     ........
```

**获取pod资源的配置清单二级级其他级别的字段**v

```
[root@k8s-master01 ~]# kubectl explain pod.metadata #查看一级字段中有哪些二级字段，字段的上下级以 "." 定义
KIND:     Pod
VERSION:  v1

RESOURCE: metadata <Object>

DESCRIPTION:
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

     ObjectMeta is metadata that all persisted resources must have, which
     includes all objects users must create.
........
```

```
[root@k8s-master01 ~]# kubectl explain pod.metadata.labels #查看二级字段中有哪些三级字段
KIND:     Pod
VERSION:  v1

FIELD:    labels <map[string]string>

DESCRIPTION:
     Map of string keys and values that can be used to organize and categorize
     (scope and select) objects. May match selectors of replication controllers
     and services. More info: http://kubernetes.io/docs/user-guide/labels
```

**字段配置的格式**

```
帮助信息中常见格式如下：
apiVersion <string>          #表示字符串类型
metadata <Object>            #表示需要嵌套多层字段
labels <map[string]string>   #表示由k:v组成的映射
finalizers <[]string>        #表示字串列表
ownerReferences <[]Object>   #表示对象列表
hostPID <boolean>            #布尔类型
priority <integer>           #整型
name <string> -required-     #如果类型后面接 -required-，表示为必填字段
```

### **第四章、创建一个配置清单实例**

#### **1.以pod为例，创建一个简单的yaml文件**

```
[root@k8s-master01 ~]# mkdir manifests
[root@k8s-master01 ~]# cd manifests/
[root@k8s-master01 manifests]# cat pod-demo.yaml 
apiVersion: v1   
kind: Pod
metadata:
  name: pod-demo
  labels:
    app: myapp        #给自己打上标签
    tier: frontend
spec:
  containers:         #创建了两个容器
  - name: nginx
    image: ikubernetes/myapp:v1
  - name: tomcat
    image: tomcat:7-alpine
[root@k8s-master01 manifests]# kubectl create -f pod-demo.yaml #使用create 子命令以yaml文件的方式启动pod
[root@k8s-master01 manifests]# kubectl get pod   #主要查看pod的状态是否支持，因为有一个以上的pod，READY段需要注意pod中的容器是否全部就绪
NAME                            READY     STATUS      RESTARTS   AGE
......
pod-demo                        2/2       Running     0          2h
```

**为了便于访问，我们再创建一个service便于外部访问测试**

```
[root@k8s-master01 manifests]# cat svc-demo.yaml 
apiVersion: v1
kind: Service      #主要类型
metadata:
  name: test-service
  labels:
    app1: nginx
    app2: tomcat
spec:
  ports:   #暴露的端口设置
  - name: nginx
    port: 80     #service的端口
    targetPort: 80    #pod上暴露的端口
    nodePort: 32080   #Node上暴露的端口，需要注意的是，Node只能暴露30000-32767之间的端口
  - name: tomcat
    port: 8080
    targetPort: 8080
    nodePort: 32088
  selector:
    app: myapp
  type: NodePort    #service 端口暴露的类型，默认是ClusterIP
[root@k8s-master01 manifests]# kubectl create -f svc-demo.yaml
```

[root@k8s-master01 manifests]# kubectl get svc -o wide #查看svc的状态
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE SELECTOR
.......
test-service  NodePort  10.108.230.27  <none>  80:32080/TCP,8080:32088/TCP  22m  app=myapp   #根据暴露的端口，加上任意集群的IP地址进行访问

#### **2.pod资源清单示例**

```
[root@k8s-master01 ~]# kubectl get pod nginx-deploy-7db697dfbd-2qh7v -o yaml  #使用 -o 参数 加yaml，可以将资源的配置以 yaml的格式输出出来，也可以使用json，输出为json格式
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: 2018-10-30T05:40:55Z
  generateName: nginx-deploy-7db697dfbd-
  labels:
    pod-template-hash: "3862538968"
    run: nginx-deploy
  name: nginx-deploy-7db697dfbd-2qh7v
  namespace: default
  ownerReferences:
  - apiVersion: extensions/v1beta1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nginx-deploy-7db697dfbd
    uid: 0eef9e1c-dbf0-11e8-8969-5254001b07db
  resourceVersion: "15622"
  selfLink: /api/v1/namespaces/default/pods/nginx-deploy-7db697dfbd-2qh7v
  uid: 5ee94f2a-dc06-11e8-8969-5254001b07db
spec:
  containers:
  - image: nginx:1.14-alpine
    imagePullPolicy: IfNotPresent
    name: nginx-deploy
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-tcwjz
      readOnly: true
  dnsPolicy: ClusterFirst
  nodeName: k8s-node02
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-tcwjz
    secret:
      defaultMode: 420
      secretName: default-token-tcwjz
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: 2018-10-30T05:40:55Z
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: 2018-10-30T05:41:06Z
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: 2018-10-30T05:40:55Z
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://b75740e5919bd975755b256c83e03b63ea95cf2307ffc606abd03b59fea6634a
    image: docker.io/nginx:1.14-alpine
    imageID: docker-pullable://docker.io/nginx@sha256:8976218be775f4244df2a60a169d44606b6978bac4375192074cefc0c7824ddf
    lastState: {}
    name: nginx-deploy
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: 2018-10-30T05:41:06Z
  hostIP: 172.16.150.214
  phase: Running
  podIP: 10.244.2.7
  qosClass: BestEffort
  startTime: 2018-10-30T05:40:55Z
```

