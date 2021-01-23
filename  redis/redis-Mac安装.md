# Redis Mac 安装

[TOC]



#### 1、没使用Homebrew

```shell
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```



#### 2、使用Homebrew安装Redis

```sh
brew install redis

==> redis
To have launchd start redis now and restart at login:
  brew services start redis
Or, if you don't want/need a background service you can just run:
  redis-server /usr/local/etc/redis.conf
```



#### 3、 查看安装及配置文件位置

- Homebrew安装的软件会默认在`/usr/local/Cellar/`路径下
- redis的配置文件`redis.conf`存放在`/usr/local/etc`路径下



#### 4、启动redis服务

```
//方式一：使用brew帮助我们启动软件
brew services start redis
==> Successfully started `redis` (label: homebrew.mxcl.redis)

//方式二
redis-server /usr/local/etc/redis.conf
//执行以下命令
redis-server
```



#### 5、查看redis服务进程

我们可以通过下面命令查看redis是否正在运行

```java
ps axu | grep redis
```



#### 6、redis-cli连接redis服务

redis默认端口号**6379**，默认**auth**为空，输入以下命令即可连接

```
redis-cli -h 127.0.0.1 -p 6379
```



#### 7、启动 redis 客户端

```java
$redis-cli
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> PING
PONG
```



#### 8、关闭redis服务

- 正确停止Redis

```
redis-cli shutdown
```

- 强行终止redis

```
sudo pkill redis-server
```



#### 9、redis.conf 配置文件详解

redis默认是前台启动，如果我们想以守护进程的方式运行（后台运行），可以在**redis.conf**中将`daemonize no`,修改成`yes`即可。