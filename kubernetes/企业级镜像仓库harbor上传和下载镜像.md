push 镜像到harbor

## 测试上传和下载镜像

### 1.修改各docker client配置

```powershell
# vim /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry 192.168.50.180
```

增加 **--insecure-registry harbor.hubait.com** 即可。
重启docker：

```shell
# systemctl daemon-reload
# systemctl  restart docker
```

或者

```shell
创建/etc/docker/daemon.json文件，在文件中指定仓库地址
# cat > /etc/docker/daemon.json << EOF
{ "insecure-registries":["192.168.50.180"] }
EOF
然后重启docker就可以。

# systemctl  restart docker
```

这样设置完成后，就不会提示我们使用https的错误了。

### 2.创建Dockerfile

```shell
# vim Dockerfile 
FROM centos:centos7.1.1503
ENV TZ "Asia/Shanghai"
```

### 3.创建镜像

```shell
# docker build -t 192.168.50.180/k8s-cluster/centos7.1:0.1 .
```

### 4.把镜像push到Harbor

```shell
# docker login 192.168.50.180
# docker push 192.168.50.180/k8s-cluster/centos7.1:0.1 #k8s-cluster 表示harbor上的项目名称
```

如果不是自己创建的镜像，记得先执行 docker tags 给镜像做tag
例如：

```shell
# docker pull busybox
# docker tag busybox:latest 192.168.50.180/k8s-cluster/busybox:latest
# docker push 192.168.50.180/k8s-cluster/busybox:latest
```