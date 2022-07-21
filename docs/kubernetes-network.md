# kubernetes网络篇

## 从docker0开始
> Linux的network bridge和network namespace是两个非常重要的概念，它们是理解Kubernetes网络工作原理的基础。本文从docker0出发，讨论network bridge的工作原理。

Linux的network bridge和network namespace是两个非常重要的概念，它们是理解Kubernetes网络工作原理的基础。本文从docker0出发，讨论network bridge的工作原理。

网络名字空间(network namespace)是Linux内核提供的一项非常重要的功能，它是网络虚拟化技术的基础，也是Kubernetes和以Docker为代表的容器技术在实现它们各自的网络时所依赖的基础。所以，要理解Kubernetes的网络工作原理，首先要从network namespace入手。

在Linux内核里，每个network namespace都有它自己的网络设置，比如像：网络接口(network interfaces)，路由表(routing tables)等。我们利用network namespace，可以把不同的网络设置彼此隔离开来。当运行多个Docker容器的时候，Docker会在每个容器内部创建相应的network namespace，从而实现不同容器之间的网络隔离。

但是光有隔离还不行，因为容器还要和外界进行网络联通，所以除了network namespace以外，另一个重要的概念是网桥(network bridge)。它是由Linux内核提供的一种链路层设备，用于在不同网段之间转发数据包。Docker就是利用网桥来实现容器和外界之间的通信的。默认情况下，Docker服务会在它所在的机器上创建一个名为docker0的网桥。


docker0是由Docker服务在启动时自动创建出来的以太网桥。默认情况下，所有Docker容器都会连接到docker0，然后再通过这个网桥来实现容器和外界之间的通信。那么，Docker具体是怎么做到这一点的呢？我们先用docker network ls命令来看一下Docker默认提供的几种网络：

```shell
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
d307261937e9        bridge              bridge              local
9a4e6a25b62e        host                host                local
53cf3a3b2e4f        none                null                local
```

我们知道，Docker容器在启动时，如果没有显式指定加入任何网络，就会默认加入到名为bridge的网络。而这个bridge网络就是基于docker0实现的。

除此以外，这里的host和none是有特殊含义的。其中，加入host网络的容器，可以实现和Docker daemon守护进程（也就是Docker服务）所在的宿主机网络环境进行直接通信；而none网络，则表示容器在启动时不带任何网络设备。








### Reference
[Kubernetes网络篇——从docker0开始](https://morningspace.github.io/tech/k8s-net-docker0/)


