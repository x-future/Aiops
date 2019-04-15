

```shell
kubectl api-versions #查看各个api 版本
kubectl explain pod #查看每个资源内容
kubectl get configmap special-config -o go-template='{{.data}}'   #查看ConfigMap的内容
kubectl label node k8s-node01 disk=ssd    #对集群中的一个节点打上标签
kubectl get nodes --show-labels #查看标签
```