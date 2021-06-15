# Ubuntu 18.04网络配置



## 文件路径

Netplan 是一款使用在终端的配置网络工具，本文介绍在 Ubuntu 18.04 系统中使用 Netplan 来配置网络，新的配置文件、网络设备名称、配置静态 IP 地址、测试配置并应用、配置 DHCP。

前言

多年以来 Linux 管理员和用户们以相同的方式配置他们的网络接口。例如，如果你是 Ubuntu 用户，你能够用桌面 GUI 配置网络连接，也可以在 /etc/network/interfaces 文件里配置。配置相当简单且可以奏效。在文件中配置看起来就像这样：

auto enp10s0

iface enp10s0 inet static

address 192.168.1.162

netmask 255.255.255.0

gateway 192.168.1.100

dns-nameservers 1.0.0.1,1.1.1.1

保存并关闭文件。使用命令重启网络：

sudo systemctl restart networking

或者，如果你使用不带 systemd 的发行版，你可以通过老办法来重启网络：

sudo /etc/init.d/networking restart

你的网络将会重新启动，新的配置将会生效。

这就是多年以来的做法。但是现在，在某些发行版上（例如 Ubuntu Linux 18.04），网络的配置与控制发生了很大的变化。不需要那个 interfaces 文件和 /etc/init.d/networking 脚本，我们现在转向使用 Netplan。Netplan 是一个在某些 Linux 发行版上配置网络连接的命令行工具。Netplan 使用 YAML 描述文件来配置网络接口，然后，通过这些描述为任何给定的呈现工具生成必要的配置选项。

我将向你展示如何在 Linux 上使用 Netplan 配置静态 IP 地址和 DHCP 地址。本文在 Ubuntu Server 18.04 系统中演示。有句忠告，你创建的 .yaml 文件中的缩进必须保持一致，否则将会失败。你不用为每行使用特定的缩进间距，只需保持一致就行了。



## 新的配置文件



打开终端窗口（或者通过 SSH 登录进 Ubuntu 服务器）。你会在 /etc/netplan 文件夹下发现 Netplan 的新配置文件。使用 cd /etc/netplan 命令进入到那个文件夹下。一旦进到了那个文件夹，也许你就能够看到一个文件：

01-netcfg.yaml

你可以创建一个新的文件或者是编辑默认文件。如果你打算修改默认文件，我建议你先做一个备份：

sudo cp /etc/netplan/01-netcfg.yaml /etc/netplan/01-netcfg.yaml.bak

备份好后，就可以开始配置了。

```shell
ubuntu@VM-0-8-ubuntu:/etc/netplan$ cd /etc/netplan/
ubuntu@VM-0-8-ubuntu:/etc/netplan$ ll
total 16
drwxr-xr-x   2 root root 4096 May 29  2019 ./
drwxr-xr-x 101 root root 4096 Apr  2 11:37 ../
-rw-r--r--   1 root root  153 Aug  8  2018 01-netcfg.yaml
-rw-r--r--   1 root root  473 Mar 16 16:39 50-cloud-init.yaml
```



## 网络设备名称

在你开始配置静态 IP 之前，你需要知道设备名称。要做到这一点，你可以使用命令 ip a，然后找出哪一个设备将会被用到：

```shell
ubuntu@VM-0-8-ubuntu:/etc/netplan$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 52:54:00:ed:dd:d2 brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.8/20 brd 172.16.15.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:feed:ddd2/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:2d:f1:c9:ec brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:2dff:fef1:c9ec/64 scope link 
       valid_lft forever preferred_lft forever
66: br-b2c57a429103: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:8b:60:71:42 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-b2c57a429103
       valid_lft forever preferred_lft forever
    inet6 fe80::42:8bff:fe60:7142/64 scope link 
       valid_lft forever preferred_lft forever
```

```shell
root@hos:~# networkctl
IDX LINK             TYPE               OPERATIONAL SETUP     
  1 lo               loopback           carrier     unmanaged 
  2 enp5s0f0         ether              routable    unmanaged 
  3 enp5s0f1         ether              no-carrier  unmanaged 
  4 enp193s0f0       ether              no-carrier  unmanaged 
  5 enp193s0f1       ether              no-carrier  unmanaged 
  6 docker0          ether              no-carrier  unmanaged 

6 links listed.
```

网卡运行状态

```shell
root@hos:~# networkctl status
●        State: routable
       Address: 192.168.1.71 on enp5s0f0
                172.17.0.1 on docker0
       Gateway: 192.168.1.1 on enp5s0f0
       
    ubuntu@VM-0-8-ubuntu:/etc/netplan$ networkctl status
●        State: routable
       Address: 172.16.0.8 on eth0
                172.17.0.1 on docker0
                172.18.0.1 on br-b2c57a429103
                fe80::5054:ff:feed:ddd2 on eth0
                fe80::42:2dff:fef1:c9ec on docker0
                fe80::42:8bff:fe60:7142 on br-b2c57a429103
       Gateway: 172.16.0.1 on eth0
           DNS: 183.60.83.19
                183.60.82.98
```



## 配置静态 IP 地址

使用命令打开原来的 .yaml 文件：

```shell
sudo nano /etc/netplan/01-netcfg.yaml
```

文件的布局看起来就像这样：

```yaml
network:
　Version: 2
　Renderer: networkd
　ethernets:
　　DEVICE_NAME:
　　　Dhcp4: yes/no
　　　Addresses: [IP/NETMASK]
　　　Gateway: GATEWAY
　　　Nameservers:
　　　　Addresses: [NAMESERVER, NAMESERVER]

```

其中：

1.DEVICE_NAME 是需要配置设备的实际名称。

2.yes/no 代表是否启用 dhcp4。

3.IP 是设备的 IP 地址。

4.NETMASK 是 IP 地址的掩码。

5.GATEWAY 是网关的地址。

6.NAMESERVER 是由逗号分开的 DNS 服务器列表。

### 这是一份 .yaml 文件的样例：

```yaml
network:
　version: 2
　renderer: networkd
　ethernets:
　　ens5:
　　dhcp4: no
　　addresses: [192.168.1.230/24]
　　gateway4: 192.168.1.254
　　nameservers:
　　　addresses: [8.8.4.4,8.8.8.8]
```

编辑上面的文件以达到你想要的效果。保存并关闭文件。

注意，掩码已经不用再配置为 255.255.255.0 这种形式。取而代之的是，掩码已被添加进了 IP 地址中。参考[在Ubuntu 18.04系统中设置静态IP的方法](https://ywnz.com/linuxjc/3046.html)一文。



## 测试配置

在应用改变之前，让我们测试一下配置。为此，使用命令：

```shell
sudo netplan try
```

上面的命令会在应用配置之前验证其是否有效。如果成功，你就会看到配置被接受。换句话说，Netplan 会尝试将新的配置应用到运行的系统上。如果新的配置失败了，Netplan 会自动地恢复到之前使用的配置。成功后，新的配置就会被使用。



**应用新的配置**

如果你确信配置文件没有问题，你就可以跳过测试环节并且直接使用新的配置。它的命令是：

```shell
sudo netplan apply
```

此时，你可以使用 ip a 看看新的地址是否正确。





## 配置 DHCP

虽然你可能不会配置 DHCP 服务，但通常还是知道比较好。例如，你也许不知道网络上当前可用的静态 IP 地址是多少。你可以为设备配置 DHCP，获取到 IP 地址，然后将那个地址重新配置为静态地址。

在 Netplan 上使用 DHCP，配置文件看起来就像这样：



```yaml
network:
　version: 2
　renderer: networkd
　ethernets:
　　ens5:
　　Addresses: []
　　dhcp4: true
　　optional: true
```

保存并退出。用下面命令来测试文件：

```shell
sudo netplan try
sudo netplan --debug apply
```

Netplan 应该会成功配置 DHCP 服务。这时你可以使用 ip a 命令得到动态分配的地址，然后重新配置静态地址。或者，你可以直接使用 DHCP 分配的地址（但看看这是一个服务器，你可能不想这样做）。

也许你有不只一个的网络接口，你可以命名第二个 .yaml 文件为 02-netcfg.yaml 。Netplan 会按照数字顺序应用配置文件，因此 01 会在 02 之前使用。根据你的需要创建多个配置文件。


**结语**

上面所讲的是使用 Netplan 配置网络的方方面面，它不同于我们以往配置网络的习惯，是一个非常大的改变，但这种配置方式非常好，值得去适应，并运用在操作中。