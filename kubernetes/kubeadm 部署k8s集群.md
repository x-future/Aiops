kubeadm 部署k8s集群

kubeadm init 拉去镜像失败

root@k8s-master-01:/home/kube# kubeadm config images list
I0405 05:53:43.252229   12612 version.go:96] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0405 05:53:43.252310   12612 version.go:97] falling back to the local client version: v1.14.0
k8s.gcr.io/kube-apiserver:v1.14.0
k8s.gcr.io/kube-controller-manager:v1.14.0
k8s.gcr.io/kube-scheduler:v1.14.0
k8s.gcr.io/kube-proxy:v1.14.0
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.10
k8s.gcr.io/coredns:1.3.1

docker pull mirrorgooglecontainers/kube-apiserver:v1.14.0

docker pull mirrorgooglecontainers/kube-controller-manager:v1.14.0

docker pull mirrorgooglecontainers/kube-scheduler:v1.14.0

docker pull mirrorgooglecontainers/kube-proxy:v1.14.0

docker pull mirrorgooglecontainers/pause:3.1

docker pull mirrorgooglecontainers/etcd:3.3.10

docker pull coredns/coredns:1.3.1

docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.8.0

打tag

docker tag mirrorgooglecontainers/kube-apiserver:v1.14.0 k8s.gcr.io/kube-apiserver:v1.14.0

docker tag mirrorgooglecontainers/kube-controller-manager:v1.14.0 k8s.gcr.io/kube-controller-manager:v1.14.0

docker tag mirrorgooglecontainers/kube-scheduler:v1.14.0 k8s.gcr.io/kube-scheduler:v1.14.0

docker tag mirrorgooglecontainers/kube-proxy:v1.14.0  k8s.gcr.io/kube-proxy:v1.14.0

docker tag mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1
docker tag mirrorgooglecontainers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
docker tag coredns/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1

docker tag  mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.8.0 k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.0

生成配置文件

kubeadm config print init-defaults > kubeadm.conf

sed -i '/^imageRepository/ s/k8s\.gcr\.io/u9nigs6v\.mirror\.aliyuncs\.com\/google_containers/g' kubeadm.conf

imageRepository:u9nigs6v.mirror.aliyuncs.com/mirrorgooglecontainers

kubeadm config images list --config kubeadm.conf
kubeadm config images pull --config kubeadm.conf



## 开始初始化

初始化之前最好先了解一下 kubeadm init 参数


--apiserver-advertise-address string
API Server将要广播的监听地址。如指定为 `0.0.0.0` 将使用缺省的网卡地址。

--apiserver-bind-port int32     缺省值: 6443
API Server绑定的端口

--apiserver-cert-extra-sans stringSlice
可选的额外提供的证书主题别名（SANs）用于指定API Server的服务器证书。可以是IP地址也可以是DNS名称。

--cert-dir string     缺省值: "/etc/kubernetes/pki"
证书的存储路径。

--config string
kubeadm配置文件的路径。警告：配置文件的功能是实验性的。

--cri-socket string     缺省值: "/var/run/dockershim.sock"
指明要连接的CRI socket文件

--dry-run
不会应用任何改变；只会输出将要执行的操作。

--feature-gates string
键值对的集合，用来控制各种功能的开关。可选项有:
Auditing=true|false (当前为ALPHA状态 - 缺省值=false)
CoreDNS=true|false (缺省值=true)
DynamicKubeletConfig=true|false (当前为BETA状态 - 缺省值=false)

-h, --help
获取init命令的帮助信息

--ignore-preflight-errors stringSlice
忽视检查项错误列表，列表中的每一个检查项如发生错误将被展示输出为警告，而非错误。 例如: 'IsPrivilegedUser,Swap'. 如填写为 'all' 则将忽视所有的检查项错误。

--kubernetes-version string     缺省值: "stable-1"
为control plane选择一个特定的Kubernetes版本。

--node-name string
指定节点的名称。

--pod-network-cidr string
指明pod网络可以使用的IP地址段。 如果设置了这个参数，control plane将会为每一个节点自动分配CIDRs。

--service-cidr string     缺省值: "10.96.0.0/12"
为service的虚拟IP地址另外指定IP地址段

--service-dns-domain string     缺省值: "cluster.local"
为services另外指定域名, 例如： "myorg.internal".

--skip-token-print
不打印出由 `kubeadm init` 命令生成的默认令牌。

--token string
这个令牌用于建立主从节点间的双向受信链接。格式为 [a-z0-9]{6}\.[a-z0-9]{16} - 示例： abcdef.0123456789abcdef

--token-ttl duration     缺省值: 24h0m0s
令牌被自动删除前的可用时长 (示例： 1s, 2m, 3h). 如果设置为 '0', 令牌将永不过期。
在master上开始初始化

[root@master] ~$ kubeadm init --kubernetes-version=v1.14.0 --pod-network-cidr=10.244.0.0/16

*注：如果中途出错可以用kubeadm reset来进行回退*

 `kubeadm token list kubeadm token create`

kubeadm join 192.168.40.151:6443 --token 5lbgw4.jxzphdvx9kus4x6h \
    --discovery-token-ca-cert-hash sha256:417475de9da5859dbfb4f416079e7b6fdf3a54090db6f97de3096e67b28e5620

自动补全功能

echo "source <(kubectl completion bash)" >> ~/.bashrc

导出镜像

#k8s-master

   $ sudo docker images

   $ sudo docker save da86e6ba6ca1 f0fad859c909 1d3d7afd77d1 > node.tar

导入镜像，打tag

 #k8s-node1

   $ sudo docker load < /tmp/node.tar

   $ sudo docker tag da86e6ba6ca1 k8s.gcr.io/pause:3.1

节点无法创建pod可以用kubeadm reset 删除节点上文件，卸载kubelet 重新安装，删除docker 镜像

生成新token重新 kubeadm join

​	