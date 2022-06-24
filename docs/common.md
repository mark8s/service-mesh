# 通用知识

## 什么是南北流量？东西流量？

南北流量： 它指的是客户端到服务器之间通信的流量，英文是NORTH-SOURTH-TRAFFIC，直接是字面翻译过来的。那为啥叫南北而不叫东西呢？因为通常画网络拓扑图时，习惯上把服务器和客户端之间画成上下方向，因此叫南北流量。

东西流量：有南北自然就有东西流量(EAST-WEST-TRAFFIC)，指的是服务器和服务器之间的流量或不同数据中心之间的网络流被称为东西流量，为啥叫东西呢？原因也一样，习惯而已，网络图拓扑图中将服务器之间的流量喜欢画在水平方向，因此叫东西流量。

![east-west](../images/east-west.png)

## nohup 是脚本后台不中断执行

你可以在Linux命令或者脚本后面增加&符号，从而使命令或脚本在后台执行，例如：.
```shell
$ ./curlsolarbookinfo2 &
```
使用&符号在后台执行命令或脚本后，如果你退出登录，这个命令就会被自动终止掉。要避免这种情况，你可以使用nohup命令，如下所示：
```shell
$ nohup ./curlsolarbookinfo2 &
```

使用ps -ef |grep curlsolarbookinfo2.sh可查看到正在运行的脚本进程
退出当前shell终端，再重新打开，使用ps -ef可以看到

脚本内容：
```shell
#!/bin/bash 

while true
do 
   curl http://bookinfo.solarmesh.cn/productpage
   sleep 15
done 

```
