# Ubuntu18.04 MySQL



## 一、安装

```shell
# 查看有没有安装MySQL
dpkg -l | grep mysql
```

```shell
#更新软件源
sudo apt-get update 
#安装mysql
sudo apt-get install mysql-server  
```

![img](安装.png)

```shell
# 启动和关闭mysql服务器：
service mysql start
service mysql stop
```

![img](启动关闭.png)



## 二、检查



```shell
# 安装完成之后可以使用如下命令来检查是否安装成功：
sudo netstat -tap | grep mysql
```

![img](状态检查.png)



```shell
# 登录mysql数据库可以通过如下命令
# -u 表示选择登陆的用户名， -p 表示登陆的用户密码，现在是mysql数据库是没有密码的，Enter password:处直接回车，就能够进入mysql数据库。
mysql -u root -p
```

![img](命令行模式.png)



```shell
# 解决利用sqoop导入MySQL中文乱码的问题（可以插入中文，但不能用sqoop导入中文）导致导入时中文乱码的原因是character_set_server默认设置是latin1 可以单个设置修改编码方式set character_set_server=;但是重启会失效，建议按以下方式修改编码方式。
mysql -u root -p

# 并查看MySQL目前设置的编码
mysql> show variables like "char%";
```



![img](乱码.png)

```shell
# 在[mysqld]下添加一行character_set_server=utf8。如下图
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
# 重启MySQL服务
service mysql restart
```

![img](/Users/sky/Github/node/node_doc/mysql/不乱码.png)

```shell
# 登录
mysql -u root -p
# 并查看MySQL目前设置的编码
# 完成编码方式的修改后，即解决了sqoop导入MySQL中文乱码的问题。至此，ubuntu系统上顺利完成安装mysql数据库。
mysql> show variables like "char%";
```

![img](不乱码正确.png)

```
# 检查mysql服务状态：
systemctl status mysql
```

![img](健康状态.png)

```shell
# 处输入刚设置的密码，回车，就能够进入mysql数据库。
mysql -u root -p命令，
Enter password:

mysql> use mysql; 
# 命令打开mysql命名的数据库，显示当前数据库的表：
mysql> show tables; 
#查询user表里的数据：
mysql> select * from user;
（user表里是mysql数据库的所有账户信息）
```



![img](/Users/sky/Github/node/node_doc/mysql/表.png)



## 三、配置外网



```shell
# 现在配置mysql允许远程访问，首先编辑 /etc/mysql/mysql.conf.d/mysqld.cnf 配置文件：
vim /etc/mysql/mysql.conf.d/mysqld.cnf
# 注释掉bind-address          = 127.0.0.1
```



![img](/Users/sky/Github/node/node_doc/mysql/设置网络.png)



```shell

mysql -u root -p
# 配置外网访问
mysql> grant all on *.* to root@'%' identified by '123456' with grant option;
grant all on *.* to root@'%' identified by '123456' with grant option;

# 配置内网访问
mysql> grant all on *.* to root@'127.0.0.1' identified by '123456' with grant option;
mysql> grant all on *.* to root@'localhost' identified by '123456' with grant option;
# 刷新权限
mysql> flush privileges;    
mysql> exit
```





## 四、外网访问 （）

> root@arstech:~# mysql -V
> mysql  Ver 8.0.23-0ubuntu0.20.04.1 for Linux on x86_64 ((Ubuntu))



>mysql8 之前的版本中加密规则是mysql_native_password,
>而在mysql8之后，加密规则是caching_sha2_password 
>
>mysql -uroot -ppassword #登录
>
>use mysql; #选择数据库
>
># 远程连接请将下方的'localhost'换成'%'
>#本地连接更改加密方式
>ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; 
>####远程连接的更改加密方式#######
>ALTER USER 'root'@'%' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
>#-----------------------#
>
>#本地连接更新用户密码
>ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '密码'; 
>####远程连接更新用户密码#######
>ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '密码'; 
>#-----------------------#
>
>FLUSH PRIVILEGES; #刷新权限