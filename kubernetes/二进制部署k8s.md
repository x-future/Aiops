二进制部署k8s

## 事前准备

`所有机器`彼此网路互通

所有防火墙与SELinux 已关闭。如CentOS：
否则后续 K8S 挂载目录时可能报错 Permission denied

```
systemctl disable --now firewalld NetworkManager
setenforce 0
sed -ri '/^[^#]*SELINUX=/s#=.+$#=disabled#' /etc/selinux/config
```

关闭 dnsmasq (可选)
linux 系统开启了 dnsmasq 后(如 GUI 环境)，将系统 DNS Server 设置为 127.0.0.1，这会导致 docker 容器无法解析域名，需要关闭它

systemctl disable --now dnsmasq

Kubernetes v1.8+要求关闭系统Swap,若不关闭则需要修改kubelet设定参数( –fail-swap-on 设置为 false 来忽略 swap on),在`所有机器`使用以下指令关闭swap并注释掉`/etc/fstab`中swap的行：

swapoff -a && sysctl -w vm.swappiness=0
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab

如果是centos的话不想升级后面的最新内核可以此时升级到保守内核去掉update的exclude即可

```
yum install epel-release -y
yum install wget git  jq psmisc socat -y
yum update -y --exclude=kernel*
```





```
# 声明集群成员信息
declare -A MasterArray otherMaster NodeArray AllNode Other
MasterArray=(['k8s-master-01']=192.168.40.152 ['k8s-master-02']=192.168.40.155
otherMaster=(['k8s-master-02']=192.168.40.155
NodeArray=(['k8s-node-01']=192.168.40.153 ['k8s-node-02']=192.168.40.154)
# 下面复制上面的信息粘贴即可
AllNode=(['k8s-master-01']=192.168.40.152 ['k8s-master-02']=192.168.40.155 ['k8s-node-01']=192.168.40.153 ['k8s-node-02']=192.168.40.154)
Other=(['k8s-master-02']=192.168.40.155 ['k8s-node-01']=192.168.40.153 ['k8s-node-02']=192.168.40.154)

export         VIP=192.168.40.200

[ "${#MasterArray[@]}" -eq 1 ]  && export VIP=${MasterArray[@]} || export API_PORT=8443
export KUBE_APISERVER=https://${VIP}:${API_PORT:=6443}

#声明需要安装的的k8s版本
export KUBE_VERSION=v1.13.5

# 网卡名
export interface=ens33

# cni
export CNI_URL="https://github.com/containernetworking/plugins/releases/download"
export CNI_VERSION=v0.7.5
# etcd
export ETCD_version=v3.2.24
```

