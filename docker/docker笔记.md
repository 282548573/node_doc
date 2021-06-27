# Docker笔记

官网：https://docs.docker.com/engine/install/ubuntu/



## 一、Doker优点

1. 固定环境：Docker是容器技术，固定依赖环境的版本+自己开发的程序，不会出现版本不一致导致的问题

2. 进程隔离：服务器自己程序挂了，结果别人的程序把内存占用完了，自己程序因为内存不足挂了；Docker进程级的隔离，互不影响

3. 环境拷贝：流量打了，部署多个服务器，镜像复制多个环境一直的容器



## 二、Docker跟虚拟机区别



1. 虚拟机缺点：

   虚拟机再运行软件时必须自带操作系统，本身小 应用要携带超大的操作系统，很笨重

   虚拟机调用资源经过很多步骤：比如内存，先向Hypervisors申请虚拟内存-虚拟物理内存-真正内存

2. Docker优点：

   不携带操作系统，Docker应用非常低

   虚拟内存->真实内存

3. 对比



## 三、安装



```shell
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```





