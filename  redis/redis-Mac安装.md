# Redis Mac 

[TOC]

## 一、Redis Mac 安装

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

```sh
# 方式一：使用brew帮助我们启动软件
brew services start redis
==> Successfully started `redis` (label: homebrew.mxcl.redis)

# 方式二
redis-server /usr/local/etc/redis.conf
# 执行以下命令
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

```shell
$redis-cli
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> PING
PONG
```



#### 8、关闭redis服务

- 正确停止Redis

```shell
redis-cli shutdown
```

- 强行终止redis

```
sudo pkill redis-server
```



#### 9、redis.conf 配置文件详解

redis默认是前台启动，如果我们想以守护进程的方式运行（后台运行），可以在**redis.conf**中将`daemonize no`,修改成`yes`即可。



## 二、使用 



```shell
# 设置键内容
127.0.0.1:6379> set name sky
OK

# 获取键内容
127.0.0.1:6379> get name
"sky"

# 获取键内容
127.0.0.1:6379> get name
"sky"

# 序列化给定 key ，并返回被序列化的值。
127.0.0.1:6379> DUMP name
"\x00\x03sky\t\x00\xc6\x12+\xd6g\xb4\xb9\x01"

# 检查给定 key 是否存在。
127.0.0.1:6379> EXISTS name
(integer) 1

# 为给定 key 设置过期时间，以秒计。
127.0.0.1:6379> EXPIRE name 1
(integer) 1

# 设置 key 的过期时间以毫秒计。
127.0.0.1:6379> PEXPIRE name 1
(integer) 1

# 时间戳(unix timestamp)格式设置 key 的过期时间
127.0.0.1:6379> Expireat name 1
(integer) 1

# 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计
127.0.0.1:6379> PEXPIREAT name 1
(integer) 1

# 当前数据库的 key 移动到给定的数据库 db 当中。
127.0.0.1:6379>  MOVE name 0
(integer) 1
 
 # 移除 key 的过期时间，key 将持久保持。
127.0.0.1:6379>  PERSIST name 
(integer) 1

# 以毫秒为单位返回 key 的剩余的过期时间。
127.0.0.1:6379>  PTTL name 
(integer) 1

# 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。
127.0.0.1:6379>  TTL name 
(integer) 1

# 从当前数据库中随机返回一个 key 
127.0.0.1:6379>  RANDOMKEY 
(integer) 1


# 修改 key 的名称
127.0.0.1:6379>  RENAME name newname
(integer) 1

# 仅当 newkey 不存在时，将 key 改名为 newkey 。
127.0.0.1:6379>  RENAMENX name newname
(integer) 1
 
 
#  返回 key 所储存的值的类型。
127.0.0.1:6379>  TYPE name 
(integer) 1

# 键集合
127.0.0.1:6379> keys *
1) "name"


# 切换数据库
127.0.0.1:6379> select 3

# 数据库大小
127.0.0.1:6379> DBSIZE
(integer) 5

# 清空数据库
127.0.0.1:6379> flushdb

# 关闭数据库
127.0.0.1:6379> shutdown

# 清空数据库
127.0.0.1:6379> FLUSHALL
OK

#查看过去时间

127.0.0.1:6379> ttl name
(integer) -2	//键不存在
(integer) -1	//键永不过期
(integer) 2		//键的到过期的剩余时间

```



## 三、性能测试

https://www.runoob.com/redis/redis-benchmarks.html

### 1、参数

![image-20210123175558363](/Users/sky/Github/node/node_doc/ redis/image-20210123175558363.png)

### 2、测试

```shell
cd /usr/local/bin
# 100个并发，1000个请求
redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 1000 
```



![image-20210123180359823](/Users/sky/Github/node/node_doc/ redis/image-20210123180359823.png)



## 四、配置文件

### 1、目录

/usr/local/etc/redis.conf

>网络配置

```shell
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
bind 127.0.0.1 ::1 #ip绑定
protected-mode yes #保护模式
port 6379 #端口配置
```



> 通用配置

```shell
daemonize no # 后台运行
supervised no # 管理守护进程的
pidfile /var/run/redis_6379.pid # 进程文件
loglevel notice # 日志级别配置
logfile ""  # 日志的文件名
databases 16 # 默认16个数据库
always-show-logo yes # 是否显示logo

```



> 快照

在规定的时间内有至少有多少个key修改就会持久化到文件.rdb.aof

```shell
save 900 1 # 若果900秒内执行至少有1个key修改就执行快照持久化操作
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes  # 持久化出错了，是否工作
rdbcompression yes # 是否压缩rdb文件，会浪费cpu资源
rdbchecksum yes # 保存rdb文件的时候检查校验
dir /usr/local/var/db/redis/ # rdb文件保存的目录
```





> REPLICATION 主从复制



> SECURITY 安全

```
# requirepass foobared //
```

![image-20210124140249715](/Users/sky/Github/node/node_doc/ redis/image-20210124140249715.png)

> CLIENTS 客户端

```
maxclients 10000  # 客户端最大连接数
```

> MEMORY MANAGEMENT 最大内存设置

```shell
maxmemory <bytes> # 最大内存设置
maxmemory-policy noeviction # 内存上限慢了采取的策略1、移除过期的key；2、
# volatile-lru -> 只对设置过期的Key进行LRU
# allkeys-lru -> 删除LRU算法的key
# volatile-lfu -> 随机删除即将过期的key
# allkeys-lfu -> 随机删除
# volatile-random -> Remove a random key having an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> 删除即将过期的
# noeviction ->永不删除，返回错误
 
```



>  APPEND ONLY MODE

```shell
appendonly no #  默认不开启 ofa 模式,默认是使用rdb持久化模式，大部分情况下，rdb是够用了
appendfilename "appendonly.aof" # 可以自定义
appendfsync everysec # 默认每秒执行一次 # appendfsync always 每次修改都会同步

```



## 五、5大数据类型

### 1、String

#### 1）命令

```shell
# 字符串追加
127.0.0.1:6379> APPEND name www
(integer) 4

# 字符串长度
127.0.0.1:6379> STRLEN name
(integer) 4

# 自加1
127.0.0.1:6379> INCR age
(integer) 4

# 自减1
127.0.0.1:6379> DECR age
(integer) 2

# 步长加10
127.0.0.1:6379> INCRBY age 10
(integer) 11

# 步长减5
127.0.0.1:6379> DECRBY age 5
(integer) 6

# 截取字符串
127.0.0.1:6379> GETRANGE name 0 1
"ab"

# 获取全部字符串
127.0.0.1:6379> GETRANGE name 0 -1
"abcdefg"

# 指定位置开始写入
127.0.0.1:6379> SETRANGE name 1 zx
(integer) 7

# 带过期时间设置(秒)
127.0.0.1:6379> SETEX name1 10 skyyyyy
OK

#存在创建失败
127.0.0.1:6379> SETNX name1 sksksksk
(integer) 1	//不存在创建成功
127.0.0.1:6379> SETNX name1 sksksksk
(integer) 0	//存在创建失败

#批量创建
127.0.0.1:6379> MSET k1 v1 k2 v2
OK

#批量过去
127.0.0.1:6379> MGET k1 k2
1) "v1"
2) "v2"

#批量原子级别的设置
127.0.0.1:6379> MSETNX k1 v1 k3 v2
(integer) 0

#返回原值再设置新值；没有就返回nil
127.0.0.1:6379> GETSET k1 v112
"v1111"
127.0.0.1:6379> GETSET k1 v113
"v112"
```

#### 2）使用场景

- 计算器
- 统计多单位的数量
- 粉丝数
- 对象缓存





### 2、列表

Redis 的列表都可以做成队列、堆栈、阻塞队列，所有队列都是l开头的



> Swwwwwwww
>
> wniiw
>
> weel
>
> Wggg wgegegwlklk









