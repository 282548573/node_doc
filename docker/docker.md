# Docker

## 一、安装

### 官网

https://docs.docker.com/engine/install/ubuntu/

```shell
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

### 查看服务状态：

```shell
ubuntu@VM-0-8-ubuntu:~/sky/dir$ systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2021-03-24 09:43:18 CST; 3 months 3 days ago
     Docs: https://docs.docker.com
 Main PID: 26621 (dockerd)
    Tasks: 12
   CGroup: /system.slice/docker.service
           └─26621 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

### 启动服务：

```shell
systemctl start docker
systemctl stop docker

systemctl enable docker
#查看状态
ubuntu@VM-0-8-ubuntu:~/sky/dir$ docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Build with BuildKit (Docker Inc., v0.5.1-docker)

Server:
ERROR: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.24/info: dial unix /var/run/docker.sock: connect: permission denied
errors pretty printing info


```



### 创建Docker组

```shell
sudo groupadd docker
sudo usermod -aG docker $USER
systemctl restart docker
```



## 二、镜像配置

### 1、腾讯云镜像配置

> 腾讯云上 pull 镜像慢的一笔，改成腾讯自己的镜像会非常顺畅，修改方法如下

创建或修改 `/etc/docker/daemon.json` 文件，并写入一下内容：

```json
{
   	"registry-mirrors": [
				"https://mirror.ccs.tencentyun.com"
  	]
}
```

重新启动 Docker 服务 。重启之前可以 `docker ps`看下自己起了哪些服务，别忘了重启

```shell
systemctl daemon-reload
service docker restart
```

`docker info`查看是否生效 在最下面有这两行即为成功

```
Registry Mirrors:
 https://mirror.ccs.tencentyun.com
```



### 2、镜像网站

```http
https://hub.docker.com
```



## 三、功能



#### docker run 

###### 普通运行

> **docker**: Docker 的二进制执行文件。
>
> **run**: 与前面的 docker 组合来运行一个容器。
>
> **ubuntu:15.10** 指定要运行的镜像，Docker 首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。
>
> **/bin/echo** "Hello world": 在启动的容器里执行的命令
>
> 以上命令完整的意思可以解释为：Docker 以 ubuntu15.10 镜像创建一个新容器，然后在容器里执行 bin/echo "Hello world"，然后输出结果。

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker run ubuntu:15.10 /bin/echo "Hello world"
Unable to find image 'ubuntu:15.10' locally
15.10: Pulling from library/ubuntu
7dcf5a444392: Pull complete 
759aa75f3cee: Pull complete 
3fa871dc8a2b: Pull complete 
224c42ae46e7: Pull complete 
Digest: sha256:02521a2d079595241c6793b2044f02eecf294034f31d6e235ac4b2b54ffc41f3
Status: Downloaded newer image for ubuntu:15.10
Hello world
```

###### 容器命令行终端

> - **-t:** 在新容器内指定一个伪终端或终端。
> - **-i:** 允许你对容器内的标准输入 (STDIN) 进行交互。
>
> 注意第二行 **root@0123ce188bd8:/#**，此时我们已进入一个 ubuntu15.10 系统的容器
>
> 退出容器 ：exit 或  CTRL+D 



```
ubuntu@VM-0-8-ubuntu:~$ docker run -i -t ubuntu:15.10 /bin/bash
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/create: dial unix /var/run/docker.sock: connect: permission denied.
See 'docker run --help'.
ubuntu@VM-0-8-ubuntu:~$ sudo docker run -i -t ubuntu:15.10 /bin/bash
root@81e26accf509:/# ll
```

###### 后台模式

> 在输出中，我们没有看到期望的 "hello world"，容器 ID

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
df6fe458e225e63117f8423200b316fe67cf7da4776c13342f9ac5a29ebbc4dc
```

###### 后台模式- 指定名字

```
docker run -itd --name ubuntu-test ubuntu:15.10 /bin/bash
```



###### 后台模式- 指定名字 端口

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker run -itd -p 5001:5000 --name ubuntu-test1 ubuntu:15.10 /bin/bash
308565f9134b01b099b05fce615ab27ac76c7bc80799153a445d866e079cfdb9
ubuntu@VM-0-8-ubuntu:~$ sudo docker ps -al
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS          PORTS                    NAMES
308565f9134b   ubuntu:15.10   "/bin/bash"   13 seconds ago   Up 12 seconds   0.0.0.0:5001->5000/tcp   ubuntu-test1
```

###### 

```shell
#-p 宿主机端口:docker端口
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker run -itd -p 127.0.0.1:5002:5000 --name ubuntu-test4 ubuntu:15.10 /bin/bash
01786b04e74327224d14d56fa96175147168cfb37a0bab081018ac2a27d16b7d
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED              STATUS              PORTS                      NAMES
01786b04e743   ubuntu:15.10   "/bin/bash"   3 seconds ago        Up 2 seconds        127.0.0.1:5002->5000/tcp   ubuntu-test4
```





###### 后台模式- 随机 端口

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker run -itd -P --name ubuntu-test1 ubuntu:15.10 /bin/bash
308565f9134b01b099b05fce615ab27ac76c7bc80799153a445d866e079cfdb9
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS          PORTS     NAMES
f6e34ee7bf31   ubuntu:15.10   "/bin/bash"   13 seconds ago   Up 12 seconds             ubuntu-test1
```





###### 后台模式- UDP

> 上面的例子中，默认都是绑定 tcp 端口，如果要绑定 UDP 端口，可以在端口后面加上 **/udp**。

```shell
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker run -itd -p 127.0.0.1:5002:5000/udp  --name ubuntu-test4 ubuntu:15.10 /bin/bash
01786b04e74327224d14d56fa96175147168cfb37a0bab081018ac2a27d16b7d
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED              STATUS              PORTS                      NAMES
01786b04e743   ubuntu:15.10   "/bin/bash"   3 seconds ago        Up 2 seconds        127.0.0.1:5002->5000/tcp   ubuntu-test4
```



###### 常用的后台模式

```shell
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker run -d -p 5002:5000/udp  --name ubuntu-test4 ubuntu:15.10 
01786b04e74327224d14d56fa96175147168cfb37a0bab081018ac2a27d16b7d

ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED              STATUS              PORTS                      NAMES
01786b04e743   ubuntu:15.10   "/bin/bash"   3 seconds ago        Up 2 seconds        127.0.0.1:5002->5000/tcp   ubuntu-test4
```



#####  **docker ps** 

> **CONTAINER ID:** 容器 ID。
>
> **IMAGE:** 使用的镜像。
>
> **COMMAND:** 启动容器时运行的命令。
>
> **CREATED:** 容器的创建时间。
>
> **STATUS:** 容器状态。
>
> 状态有7种：
>
> - created（已创建）
> - restarting（重启中）
> - running 或 Up（运行中）
> - removing（迁移中）
> - paused（暂停）
> - exited（停止）
> - dead（死亡）
>
> **PORTS:** 容器的端口信息和使用的连接类型（tcp\udp）。
>
> **NAMES:** 自动分配的容器名称。

```shell
# 运行中的镜像
ubuntu@VM-0-8-ubuntu:~$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
df6fe458e225   ubuntu:15.10   "/bin/sh -c 'while t…"   42 seconds ago   Up 42 seconds             vigilant_nightingale
1837c3be4148   ubuntu:15.10   "/bin/bash"              19 minutes ago   Up 19 minutes             sleepy_lamarr

# 全部镜像
ubuntu@VM-0-8-ubuntu:~$ sudo docker ps -a

```



##### docker start

> 启动已停止的容器

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS                          PORTS     NAMES
d42148dfc398   ubuntu:15.10   "/bin/bash"              About a minute ago   Exited (0) About a minute ago  

ubuntu@VM-0-8-ubuntu:~$ sudo docker start d42148dfc398

ubuntu@VM-0-8-ubuntu:~$ sudo docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                        PORTS     NAMES
d42148dfc398   ubuntu:15.10   "/bin/bash"              2 minutes ago    Up 3 seconds                            nervous_solomon
31dd4213e5cd   ubuntu:15.10   "/bin/bash"              4 minutes ago    Up 4 minutes                            cranky_feynman
77ddefad12b8   ubuntu:15.10   "/bin/bash"              5 minutes ago    Exited (0) 5 minutes ago                nervous_feistel

```



##### docker restart 

> 重启容器

```shell
sudo docker restart d42148dfc398
d42148dfc398
```



##### docker logs 

> 查看容器内的标准输出

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker logs df6fe458e225e63117f8423200b316fe67cf7da4776c13342f9ac5a29ebbc4dc
hello world
hello world
```



##### docker stop

> 停止容器:  CONTAINER ID 或者 NAMES

```
ubuntu@VM-0-8-ubuntu:~$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
df6fe458e225   ubuntu:15.10   "/bin/sh -c 'while t…"   5 minutes ago    Up 5 minutes              vigilant_nightingale
1837c3be4148   ubuntu:15.10   "/bin/bash"              25 minutes ago   Up 25 minutes             sleepy_lamarr
ubuntu@VM-0-8-ubuntu:~$ sudo docker stop df6fe458e225
df6fe458e225
ubuntu@VM-0-8-ubuntu:~$ sudo docker stop sleepy_lamarr
df6fe458e225
```



##### docker pull

> 获取镜像 软件:版本号

```shell
sudo docker pull ubuntu:15.10 
sudo docker run -it ubuntu:15.10 /bin/bash
```





##### docker attach

>  如果从这个容器退出，会导致容器的停止。

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS         PORTS     NAMES
31dd4213e5cd   ubuntu:15.10   "/bin/bash"   8 minutes ago   Up 8 minutes             cranky_feynman

ubuntu@VM-0-8-ubuntu:~$ sudo docker attach d42148dfc398
root@d42148dfc398:/# exit;

ubuntu@VM-0-8-ubuntu:~$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS         PORTS     NAMES
```



##### docker exec

> **注意：** 如果从这个容器退出，容器不会停止，这就是为什么推荐大家使用 **docker exec** 的原因。
>
> 更多参数说明请使用 **docker exec --help** 命令查看。

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker exec -it ubuntu-test /bin/bash
root@a678d89378df:/# 
```

##### docker export

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker export a678d89378df > ubuntu_sky.tar
```



##### docker import

本地导入

```shell
ubuntu@VM-0-8-ubuntu:~$ cat ubuntu_sky.tar | sudo  docker import - ubuntu_sky_b:v1
sha256:a4cc2cc0d648fc0de08dc08cd57495ead6ea57c5952b2a25f590ad0114c5279a
ubuntu@VM-0-8-ubuntu:~$ sudo docker import  ubuntu_sky.tar ubuntu_sky1:v4 
ubuntu@VM-0-8-ubuntu:~$ sudo docker images 
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
ubuntu_sky1    v4        9b37a67d16dd   11 seconds ago   119MB
```

Url导入

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker import http://example.com/exampleimage.tgz example/imagerepo
```



##### docker container prune 

> 清理掉所有处于终止状态的容器

```
ubuntu@VM-0-8-ubuntu:~$ sudo docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
77ddefad12b843f884d230cebd19deb8fae45b19a13994f6096f7d828d95ec2b
df6fe458e225e63117f8423200b316fe67cf7da4776c13342f9ac5a29ebbc4dc
5f3ec37ee9d46ab6b8e263b9c9a702d8a5039bf32ed43d17de9dc4591a85206e
9d0b6e08d57974cb512fbf4a71c18f866be413bf5f4ea23ba76d02f7b02cbcf0
ba1d89450e058db16a0aa1e82e82f7b51c35f05626b0a0066aa51f419920ec39
1837c3be41489f2740cd2e5ab6c1a73a512d0a2acdf709e636d4e75b3e5961db
ba2f47a73b068a5ffcccde556423a2d83b6d44b3eedf72588216a58c92f3384a
81e26accf5097dbc16ed5cb9a5c987ce9315532dca65790a5d7f695041900e02
00d03d951888f001b6d77f532a1a831eacb6783bdf7acdb8e0e814580517af10
7dbcf81c32c4a68d36d458f9ab81301d6179bc9cba1293e0fdf83444c8a943cb
e946d486a73f5dc02dff34a24d55e8bcc8646389e7d3378b65022e63a489dc95
```

##### docker port 

> 通过 **docker ps** 命令可以查看到容器的端口映射，**docker** 还提供了另一个快捷方式 **docker port**，使用 **docker port** 可以查看指定 （ID 或者名字）容器的某个确定端口映射到宿主机的端口号。

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker port 308565f9134b01b099b05fce615ab27ac76c7bc80799153a445d866e079cfdb9
5000/tcp -> 0.0.0.0:5001
```



##### docker logs

> docker logs [ID或者名字] 可以查看容器内部的标准输出。

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker logs -f 308565f9134b01b099b05fce615ab27ac76c7bc80799153a445d866e079cfdb9
```



##### docker top

> 我们还可以使用 docker top 来查看容器内部运行的进程

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker ps 
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS         PORTS                    NAMES
308565f9134b   ubuntu:15.10   "/bin/bash"   6 minutes ago   Up 6 minutes   0.0.0.0:5001->5000/tcp   ubuntu-test1
ubuntu@VM-0-8-ubuntu:~$ sudo docker top 308565f9134b
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                7456                7425                0                   17:50               pts/0               00:00:00            /bin/bash
```

##### docker inspect

> 使用 **docker inspect** 来查看 Docker 的底层信息。它会返回一个 JSON 文件记录着 Docker 容器的配置和状态信息。

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker inspect 308565f9134b
[
    {
        "Id": "308565f9134b01b099b05fce615ab27ac76c7bc80799153a445d866e079cfdb9",
        "Created": "2021-03-24T09:50:30.927432083Z",
        "Path": "/bin/bash",
        "Args": [],
```

## 四、镜像

#####  docker images

> 来列出本地主机上的镜像。
>
> - **REPOSITORY：**表示镜像的仓库源
>
> - **TAG：**镜像的标签
>
> - **IMAGE ID：**镜像ID
>
> - **CREATED：**镜像创建时间
>
> - **SIZE：**镜像大小
>
>   同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，如 ubuntu 仓库源里，有 15.10、14.04 等多个不同的版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。
>
>   所以，我们如果要使用版本为15.10的ubuntu系统镜像来运行容器时，命令如下：

```
ubuntu@VM-0-8-ubuntu:~$ sudo docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
ubuntu_sky1    v4        9b37a67d16dd   49 minutes ago   119MB
<none>         <none>    8ef8c03f613c   54 minutes ago   119MB
<none>         <none>    67dff1469ac2   54 minutes ago   119MB
<none>         <none>    0f2e1c22ca75   54 minutes ago   119MB
ubuntu_sky_b   v1        a4cc2cc0d648   55 minutes ago   119MB
hello-world    latest    d1165f221234   2 weeks ago      13.3kB
ubuntu         15.10     9b9cb95443b5   4 years ago      137MB
```

#####  docker pull

> 当我们在本地主机上使用一个不存在的镜像时 Docker 就会自动下载这个镜像。如果我们想预先下载这个镜像，我们可以使用 docker pull 命令来下载它。

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker pull ubuntu:15.10
15.10: Pulling from library/ubuntu
Digest: sha256:02521a2d079595241c6793b2044f02eecf294034f31d6e235ac4b2b54ffc41f3
Status: Image is up to date for ubuntu:15.10
docker.io/library/ubuntu:15.10
```

##### docker search

> 我们可以从 Docker Hub 网站来搜索镜像，Docker Hub 网址为： **https://hub.docker.com/**
>
> 我们也可以使用 docker search 命令来搜索镜像。比如我们需要一个 httpd 的镜像来作为我们的 web 服务。我们可以通过 docker search 命令搜索 httpd 来寻找适合我们的镜像。
>
> **NAME:** 镜像仓库源的名称
>
> **DESCRIPTION:** 镜像的描述
>
> **OFFICIAL:** 是否 docker 官方发布
>
> **stars:** 类似 Github 里面的 star，表示点赞、喜欢的意思。
>
> **AUTOMATED:** 自动构建。

```
ubuntu@VM-0-8-ubuntu:~$ sudo docker search httpd
NAME                                    DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
httpd                                   The Apache HTTP Server Project                  3419      [OK]       
centos/httpd-24-centos7                 Platform for running Apache httpd 2.4 or bui…   37                   
centos/httpd                                                                            33                   [OK]
```



##### docker rm

运行时删除



```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker rm d42148dfc398
Error response from daemon: You cannot remove a running container d42148dfc3989073ae77039fec49852080eed07acc17c898fcfdac0311d737bc. Stop the container before attempting removal or force remove
ubuntu@VM-0-8-ubuntu:~$ sudo docker rm -f d42148dfc398
d42148dfc398
```



```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS          PORTS     NAMES
a678d89378df   ubuntu:15.10   "/bin/bash"   33 minutes ago   Up 31 minutes             ubuntu-test
d42148dfc398   ubuntu:15.10   "/bin/bash"   6 hours ago      Up 5 hours                nervous_solomon


```

##### 

##### docker rmi

> 镜像删除使用 **docker rmi** 命令，比如我们删除 hello-world 镜像：

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker images 
REPOSITORY     TAG       IMAGE ID       CREATED             SIZE
ubuntu_sky1    v4        9b37a67d16dd   About an hour ago   119MB
<none>         <none>    8ef8c03f613c   About an hour ago   119MB
<none>         <none>    67dff1469ac2   About an hour ago   119MB
<none>         <none>    0f2e1c22ca75   About an hour ago   119MB
ubuntu_sky_b   v1        a4cc2cc0d648   About an hour ago   119MB
hello-world    latest    d1165f221234   2 weeks ago         13.3kB
ubuntu         15.10     9b9cb95443b5   4 years ago         137MB
ubuntu@VM-0-8-ubuntu:~$ sudo docker rmi 9b37a67d16dd
Untagged: ubuntu_sky1:v4
Deleted: sha256:9b37a67d16ddf21452508466fdfc8f94dbc2f1252587efeca1ebebf304a9758d
```

##### docker commit

> 此时 ID 为 e218edb10161 的容器，是按我们的需求更改的容器。我们可以通过命令 docker commit 来提交容器副本。
>
> - **-m:** 提交的描述信息
> - **-a:** 指定镜像作者
> - **e218edb10161：**容器 ID
> - **runoob/ubuntu:v2:** 指定要创建的目标镜像名

```shell
ubuntu@VM-0-8-ubuntu:~$ sudo docker run -t -i ubuntu:15.10 /bin/bash
root@7003f6b71a26:/# docker commit -m="has update" -a="runoob" 7003f6b71a26 runoob/ubuntu:v2
root@7003f6b71a26:/# exit
ubuntu@VM-0-8-ubuntu:~$ sudo docker commit -m="has update" -a="runoob" 7003f6b71a26 runoob/ubuntu:v2
sha256:0d9a1fddefd149fefdb04ceb854052d74b7ceaa92094d5d9c3456c89d0e9d5cf
ubuntu@VM-0-8-ubuntu:~$ sudo docker images
REPOSITORY      TAG       IMAGE ID       CREATED              SIZE
runoob/ubuntu   v2        0d9a1fddefd1   About a minute ago   137MB
```

##### docker build

> 编写 Dockerfile 文件

```dockerfile
FROM    centos:6.7
MAINTAINER      Fisher "fisher@sudops.com"

RUN     /bin/echo 'root:123456' |chpasswd
RUN     useradd runoob
RUN     /bin/echo 'runoob:123456' |chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
EXPOSE  22
EXPOSE  80
CMD     /usr/sbin/sshd -D
```

> - **-t** ：指定要创建的目标镜像名
> - **.** ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径
>
> 使用docker images 查看创建的镜像已经在列表中存在,镜像ID为860c279d2fec

```shell
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker build -t sky_test .
Sending build context to Docker daemon  10.75kB
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker images
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
sky_test        latest    3c1d3f62be55   17 seconds ago   191MB
```

##### docker tag

> 我们可以使用 docker tag 命令，为镜像添加一个新的标签。
>
> docker tag 镜像ID，这里是 3c1d3f62be55 ,用户名称、镜像源名(repository name)和新的标签名(tag)。
>
> 使用 docker images 命令可以看到，ID为3c1d3f62be55的镜像多一个标签。

```shell
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker tag 3c1d3f62be55 sky_test33:v3
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker images
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
sky_test33      v3        3c1d3f62be55   6 minutes ago    191MB
sky_test        latest    3c1d3f62be55   6 minutes ago    191MB
```



##### docker load -i

> 镜像导入



## 五、容器连接

> 端口映射并不是唯一把 docker 连接到另一个容器的方法。
>
> docker 有一个连接系统允许将多个容器连接在一起，共享连接信息。
>
> docker 连接会创建一个父子关系，其中父容器可以看到子容器的信息。

#### docker network

> **-d**：参数指定 Docker 网络类型，有 bridge、overlay。
>
> 其中 overlay 网络类型用于 Swarm mode，在本小节中你可以忽略它

```shell
docker network create -d bridge test-net
```

```shell
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker network create -d bridge test-net
b2c57a429103437b67ef940a3e3ffd391ce4259ed32787ad74140bec07a6dcda

ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker network ls
NETWORK ID     NAME       DRIVER    SCOPE
eeb60cae0990   bridge     bridge    local
ec86affc3f30   host       host      local
282f9f90ee52   none       null      local
b2c57a429103   test-net   bridge    local
```



```shell
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker run -itd --name test1 --network test-net ubuntu /bin/bash
383978d0fc3edbb6ccbb484523b84c73ecaa6864faee6db2950b44442ad81514
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS          PORTS                      NAMES
383978d0fc3e   ubuntu   "/bin/bash"   6 seconds ago    Up 4 seconds                               test1

ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker run -itd --name test3 --network test-net ubuntu /bin/bash
866d77f9510954fdeac0b6cb852ed3cfbffc01e2891edf49a60cdd8f246f3fda
ubuntu@VM-0-8-ubuntu:~/sky$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED              STATUS              PORTS                      NAMES
866d77f95109   ubuntu   "/bin/bash"   6 seconds ago        Up 4 seconds                                   test3

root@383978d0fc3e:/# sudo docker exec -it test1 /bin/bash
root@383978d0fc3e:/# apt-get update
root@383978d0fc3e:/# apt install iputils-ping

root@383978d0fc3e:/# sudo docker exec -it test2 /bin/bash
root@383978d0fc3e:/# apt-get update
root@383978d0fc3e:/# apt install iputils-ping

root@383978d0fc3e:/# ping test2

```

