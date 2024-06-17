# docker网络类型

## 四种网络类型

- Bridge contauner  桥接式网络模式(默认)
-  Host(open) container  开放式网络模式，和宿主机共享网络
-  Container(join) container  联合挂载式网络模式，和其他容器共享网络
-  None(Close) container  封闭式网络模式，不为容器配置网络



## 实践准备

先准备好一个镜像（Base image: `centos7` + `net-tools` + `iputils-ping` +  `curl`）

```shell
$ docker pull centos:7
$ docker run --name="centos" -it centos:7
[root@xxxxx /]# yum install net-tools iputils-ping curl
[root@xxxxx /]# exit
$ docker commit centos centos-net
```



## None网络

在run容器的时候，我们可以使用`--network`选项来设置容器的网络类型

```shell
docker run -it --network=none centos-net
```

**none网络**除了有一个本地环回`lo`（`loopback`）网络之外，就没有其他的网络了

```shell
[root@xxxx /]# ifconfig
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 8  bytes 672 (672.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 672 (672.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[root@xxxx /]# ping baidu.com
ping: baidu.com: Name or service not known
```

**本地回环**地址指的是以127开头的地址（127.0.0.1 – 127.255.255.254），通常用127.0.0.1来表示。

127.0.0.1，通常被称为本地回环地址(Loop back address)，不属于任何一个有类别地址类。它代表设备的本地虚拟接口，所以默认被看作是永远不会宕掉的接口。在windows操作系统中也有相似的定义，所以通常在不安装网卡前就可以ping通这个本地回环地址。一般都会用来检查本地网络协议、基本数据接口等是否正常的。

其主要作用有两个：

一是测试本机的网络配置，能PING通127.0.0.1说明本机的网卡和`IP`协议安装都没有问题；
另一个作用是某些SERVER/CLIENT的应用程序在运行时需调用服务器上的资源



## bridge网络

可以将之前的`centos-net`容器删除，重新创建一个bridge网络的容器。当然，这样删除之后，原来容器内的数据都会消失。容器的设计理念是“一次性使用”：容器应当是无状态的，所有持久数据应存储在外部卷中。

如果想仅仅修改网络连接类型，还可以将这个容器build为一个新的image，然后用bridge网络创建容器。

```shell
$ docker commit centos-net centos-net-none-bridge
sha256:4912e3fdfbbbcde782e45e409a2178c89bb25442ceea23138456515f38594d34
$ docker images
REPOSITORY               TAG       IMAGE ID       CREATED          SIZE
centos-net-none-bridge   latest    4912e3fdfbbb   4 seconds ago    460MB
centos-net               latest    f6467799f8a6   43 minutes ago   460MB
centos                   7         b5b4d78bc90c   4 years ago      203MB
hello-world              latest    48b5124b2768   7 years ago      1.84kB
$ docker run -it --name="centos-bridge-based-none" --network=bridge centos-net-none-bridge
[root@xxx /]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 17x.xx.0.2  netmask 255.255.0.0  broadcast 17x.xx.255.255
        ether 02:42:xx:xx:00:xx  txqueuelen 0  (Ethernet)
        RX packets 23  bytes 3681 (3.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```



## host网络

这个网络和宿主机网络完全相同。不需要进行任何的端口映射。


## Docker DNS服务器

可以直接`ping`容器名称，Docker自动将其转换为对应`ip`地址。
```shell
$ docker run --name=test1 centos-net
$ docker run -it --name=test2 centos-net
[root@xxx\] ping test1
# success!
```
注：要放到同一网络下。


## 容器与外部网络通信

容器会有一个虚拟`NAT`进行内网`ip`与公网`ip`的转换，但是这种方式只能解决容器主动向外发送数据。
为了让容器能够接受外部网络发来的数据包，可以采用端口映射的方式进行通信。

```shell
docker run -d -p 80:80 nginx
```

此时如果主机80端口接收到数据，会直接转发给相应容器。
