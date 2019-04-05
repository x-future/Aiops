k8s资源限制

1cpu =1000m

cpu.limits 最大限制

cpu.requests 最低保障

都在resources里面

QOS:

guranteed:

同时设置CPU和内存的requests 和limits 

cpu.limits=cpu.requests

memory.limits=memory.request

Burstable：

至少有一个容器设置CPU或内存资源的requests属性

BestEffort：没有任何一个

pipework