kubeadm 部署v1.14.0

关闭防火墙

systemctl stop firewalld && systemctl disable firewalld

关闭selinux

setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

关闭swap：

swapoff -a 
$ 永久
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

修改hosts文件,添加如下内容
[vm@k8s-master-01 ~]$ cat /etc/hosts

192.168.50.174 k8s-master-01
192.168.50.175 k8s-node-01

修改内核参数

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
modprobe br_netfilter

安装组件

wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

sudo cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

更新缓存

 sudo yum clean all && sudo yum makecache

卸载老的版本

```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  查看docker版本：

sudo yum list docker-ce.x86_64 --showduplicates | sort -r
```

```
$ sudo yum install docker-ce -y
```

安装kubeadm kubectl kubelet 

```bash
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

阿里docker镜像加速器

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://r6ao1ipx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

sudo systemctl enable docker

sudo systemctl enable kubelet

由于被墙需要提前拉取相关镜像

sudo docker pull mirrorgooglecontainers/kube-apiserver:v1.14.0

sudo docker pull mirrorgooglecontainers/kube-controller-manager:v1.14.0

sudo docker pull mirrorgooglecontainers/kube-scheduler:v1.14.0

sudo docker pull mirrorgooglecontainers/kube-proxy:v1.14.0

sudo docker pull mirrorgooglecontainers/pause:3.1

sudo docker pull mirrorgooglecontainers/etcd:3.3.10

sudo docker pull coredns/coredns:1.3.1

sudo docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1

打tag

sudo docker tag mirrorgooglecontainers/kube-apiserver:v1.14.0 k8s.gcr.io/kube-apiserver:v1.14.0

sudo docker tag mirrorgooglecontainers/kube-controller-manager:v1.14.0 k8s.gcr.io/kube-controller-manager:v1.14.0

sudo docker tag mirrorgooglecontainers/kube-scheduler:v1.14.0 k8s.gcr.io/kube-scheduler:v1.14.0

sudo docker tag mirrorgooglecontainers/kube-proxy:v1.14.0  k8s.gcr.io/kube-proxy:v1.14.0

sudo docker tag mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1
sudo docker tag mirrorgooglecontainers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
sudo docker tag coredns/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1

sudo docker tag  mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1

部署master

sudo kubeadm init --kubernetes-version=v1.14.0 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.50.176

添加用户环境

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

下载网络插件镜像quay.io/coreos/flannel:v0.11.0-amd64

sudo docker pull quay.io/coreos/flannel:v0.11.0-amd64

下载yaml 文件

wget <https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml>

启动flannel：kubectl apply -f  kube-flannel.yml

或者sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml

节点加入, 加入之前先要拉取好对应的镜像，可以从master导过来

sudo  kubeadm join 192.168.50.176:6443 --token dvdte6.kmtx8uf999ctun8x \
    --discovery-token-ca-cert-hash sha256:c68ee8fe61ef63a613c11eece6d43a10cd92f9282be9665b9c3e1f8dce29d6d3 

部署dashboard

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml

创建admin账户配置文件：kubernetes-dashboard-admin.rbac.yaml

---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-admin
  namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard-admin
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:

- kind: ServiceAccount
    name: kubernetes-dashboard-admin
    namespace: kube-system

---------------------

执行：kubectl create -f kubernetes-dashboard-admin.rbac.yaml



修改dashboard 暴露的类型

 kubectl -n kube-system edit service kubernetes-dashboard

type: NodePort

查看kubernete-dashboard-admin的token：

kubectl -n kube-system get secret | grep kubernetes-dashboard-admin

 kubectl describe -n kube-system secret/kubernetes-dashboard-admin-token-8fjfj