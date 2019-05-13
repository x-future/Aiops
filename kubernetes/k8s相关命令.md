

```shell
#http://docs.kubernetes.org.cn/477.html 命令集
#基础命令
kubectl api-versions #查看各个api 版本
kubectl explain pod #查看每个资源内容
kubectl create -f ./pod.json #通过pod.json文件创建一个pod
kubectl run nginx --image=nginx --replicas=5 #启动nginx实例，设置副本数5 可以加--dry-run只是打印而不时间运行
kubectl edit svc/docker-registry #编辑svc
kubectl get configmap special-config -o go-template='{{.data}}' #查看ConfigMap的内容
kubectl get nodes --show-labels #查看标签
kubectl delete rc --cascade=false #删除RC不删除pod
kubectl expose rc nginx --port=80 --target-port=8000 #为RC的nginx创建service，并通过Service的80端口转发至容器的8000端口上。
kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000

#更新升级命令集
升级deployment
kubectl set image deployment/nginx-deployment nginx=nginx:1.12.1

查询升级状态
kubectl rollout status deployment/nginx-deployment

查询升级历史
kubectl rollout history deploy/nginx-deployment

弹性扩/缩容
kubectl scale deployment nginx-deployment --replicas=10

升级/回滚
kubectl set resources deployment nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
kubectl rollout history deployment/nginx-deployment --revision=2
kubectl rollout undo deployment/nginx-deployment --to-revision=2

暂停/恢复
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deploy/nginx-deployment

弹性、按比例扩/缩容
kubectl scale deployment nginx-deployment --replicas=10
kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80 #需要和HPA联动 maxSurge=3; 25%默认  maxUnavailable=2; 25%默认 扩缩容速度（流控）

滚动更新
kubectl rolling-update redis-master -f redis-master-controller-v2.yaml #配置文件滚动升级
kubectl rolling-update redis-master --image=redis-master:2.0 #命令升级
kubectl rolling-update redis-master --image=redis-master:1.0 --rollback #pod版本回滚

使用（patch）补丁修改、更新资源的字段
kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}'
kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"kubernetes-serve-hostname","image":"new image"}]}}'

#集群管理相关的命令集
kubectl cluster-info #查看集群情况
kubectl certificate    #修改 certificate 资源.
kubectl cordon node-name #标记 node 为 unschedulable
kubectl uncordon node-name #标记 node 为 schedulable
kubectl drain node-name #设定node进入维护模式
kubectl taint nodes k8s-master-01 env=joint:NoSchedule #更新一个或者多个 node 上的 taints(调度策略)
查看节点lable
kubectl get node --show-labels
kubeclt get node -o yaml |grep A 11 labels
kubectl label node k8s-node01 disk=ssd #对集群中的一个节点打上标签

#故障检测及调试相关命令集
kubectl get events #查看事件
kubectl describe node 192.168.32.132   #显示一个指定 resource 或者 group 的 resources 详情
kubectl logs xxx -n kube-system  #输出容器在 pod 中的日志
journalctl -u kubelet -f #查看日志
```

