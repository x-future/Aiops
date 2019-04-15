# 使用Kubespray部署Kubernetes1.14.0集群

## 主机规划

| IP             | 作用          |
| :------------- | ------------- |
| 192.168.40.152 | k8s-master-01 |
| 192.168.40.153 | k8s-node-01   |
| 192.168.40.154 | k8s-node-02   |
| 192.168.40.155 | k8s-master-02 |

## 更新yum
```shell
yum update -y
```

## 更新内核

> 更新内核为 4.4.x , CentOS 默认为3.10.x

```shell
# 导入 Key

rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

# 安装 Yum 源

rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

# 更新 kernel

yum --enablerepo=elrepo-kernel install -y kernel-lt kernel-lt-devel 

# 配置 内核优先
#查看当前default的entry
awk -F \' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
grub2-editenv list
#修改默认entry
grub2-set-default 0

# 重启系统

reboot

```

## 初始化环境

所有机器上添加
```shell
 vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.40.152 k8s-master-01
192.168.40.153 k8s-node-01
192.168.40.154 k8s-node-02
192.168.40.155 k8s-master-02
```

## 配置ssh key 认证

```shell
# 确保本机也可以ssh 连接，否则下面部署失败

ssh-keygen -t rsa -N ""

ssh-copy-id -i /root/.ssh/id_rsa.pub 192.168.40.152

ssh-copy-id -i /root/.ssh/id_rsa.pub 192.168.40.153

ssh-copy-id -i /root/.ssh/id_rsa.pub 192.168.40.154

ssh-copy-id -i /root/.ssh/id_rsa.pub 192.168.40.155
```

## 增加内核配置

```shell
vi /etc/sysctl.conf

# docker
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1

sysctl -p
```

# kubespray

## 安装依赖

```shell
# 安装 centos 额外的yum源
rpm -ivh https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm

# make 缓存
yum clean all && yum makecache

# 安装 git
yum -y install git

# ansible 必须 >= 2.7
# 安装 软件
yum install -y python-pip python36 python-netaddr python36-pip ansible
```

## 下载源码

```shell
# git clone 源码

cd /opt/

git clone https://github.com/kubernetes-sigs/kubespray
```

## 安装 kubespray 依赖

```shell
cd /opt/kubespray

# 安装依赖

pip install -r requirements.txt


Successfully installed MarkupSafe-1.1.0 hvac-0.7.1 jinja2-2.10 netaddr-0.7.19 pbr-5.1.1
```

## 配置 kubespray

### hosts 配置

```shell

# 复制一份 自己的配置

cd /opt/kubespray

cp -rfp inventory/sample inventory/jicki

declare -a IPS=(192.168.40.152 192.168.40.153 192.168.40.154 192.168.40.155)

CONFIG_FILE=inventory/mycluster/hosts.ini python3 contrib/inventory_builder/inventory.py ${IPS[@]}

# 修改配置 hosts.ini

cd /opt/kubespray/inventory/jicki

rm -rf hosts.ini


vi hosts.ini

[all]
kubernetes-1 ansible_host=kubernetes-1 ansible_port=32 ip=192.168.0.247 etcd_member_name=etcd1
kubernetes-2 ansible_host=kubernetes-2 ansible_port=32 ip=192.168.0.248 etcd_member_name=etcd2
kubernetes-3 ansible_host=kubernetes-3 ansible_port=32 ip=192.168.0.249 etcd_member_name=etcd3

[kube-master]
kubernetes-1
kubernetes-2

[etcd]
kubernetes-1
kubernetes-2
kubernetes-3

[kube-node]
kubernetes-2
kubernetes-3

[k8s-cluster:children]
kube-master
kube-node

[calico-rr]
```