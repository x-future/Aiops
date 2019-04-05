kubernetes  架构概述

docker-compose 更加适用于单机编排

namespace 隔离出来了不同的space ，实现容器间的隔离

cgroups 控制进程实现，资源限制 多对多关系 限制容器资源的使用

chroot 文件系统的隔离

pid 容器有自己独立的进程表和1号进程

net 容器有自己独立的network info

ipc 在ipc通信时候，需要加入额外信息来表示进程

mnt 每个容器有自己唯一的目录挂载

utc 每个容器有独立的Hostname 和domain

docker 使用aufs来实现分层的文件系统管理

只读部分定义为Image 可写部分是container

Image类似一个单链表系统，每个image包含一个指向parent image指针

没有parent image的image 是base image

![1551019979569](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1551019979569.png)

![1551020907358](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1551020907358.png)

![1551020937321](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1551020937321.png)

![1551020952297](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1551020952297.png)



![1551020992320](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1551020992320.png)



consul 服务发现

fig



linux机制

![1551018781192](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1551018781192.png)





uninxfs

