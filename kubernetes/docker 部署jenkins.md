docker 部署jenkins

添加docker-ce.repo

```shell
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

yum -y install docker-ce
```

ubuntu 下安装命令

```shell
sudo apt install docker.io
```

启动docker服务
```shell
$sudo systemctl start docker

$ sudo systemctl enable docke
```

查看是否安装成功

```shell
$ docker -v
```

下载jenkins镜像

```shell
docker pull jenkins
```

创建文件夹

```shell
mkdir /home/jenkins
```

​          

 给uid为1000的权限

```shell
chown -R 1000:1000 jenkins/  
```

 

启动jenkins容器

```shell
docker run -itd -p 9090:8080 -p 50000:50000 --name jenkins --privileged=true  -v /home/jenkins:/var/jenkins_homejenkins:latest
```



ubuntu系统会失败

```shell
usermod -g docker jenkins

sudo docker run -itd -p 9090:8080 -p 50000:50000 --name jenkins --privileged=true  -u root -v /home/jenkins:/var/jenkins_home jenkins:latest
```

安装docker-compose

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

> ```
> sudo chmod +x /usr/local/bin/docker-compose
> ```
>
> ```
> sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
> ```

