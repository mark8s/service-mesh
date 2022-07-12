# istio multiple cluster

## 部署模型

当您将 Istio 用于生产环境部署时，需要回答一系列的问题。 网格将被限制在单个 集群 中还是分布在多个集群中？ 是将所有服务都放置在单个完全连接的网络中，还是需要网关来跨多个网络连接服务？ 
是否存在单个控制平面（可能在集群之间共享），或者是否部署了多个控制平面以确保高可用（HA）？ 如果要部署多个集群（更具体地说是在隔离的网络中），是否要将它们连接到单个多集群服务网格中， 还是将它们联合到一个 多网格 部署中？

所有这些问题，都代表了 Istio 部署的独立配置维度。

1.单一或多个集群

2.单一或多个网络

3.单一或多控制平面

4.单一或多个网格

所有组合都是可能的，尽管某些组合比其他组合更常见，并且某些组合显然不是很有趣（例如，单一集群中有多个网格）。

在涉及多个集群的生产环境部署中，部署可能使用多种模式。 例如，基于 3 个集群实现多控制平面的高可用部署，您可以通过使用单一控制平面部署 2 个集群，然后再添加第 3 个集群和 第 2 个控制平面来实现这一点，最后，再将所有 3 个集群配置为共享 2 个控制平面，以确保所有集群都有 2 个控制源来确保 HA。

如何选择正确的部署模型，取决于您对隔离性、性能和 HA 的要求。

## 安装
### 准备
1.两个k8s集群，配置如下
```shell
➜  ~ kubectl get no -owide
NAME        STATUS   ROLES                  AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
master-48   Ready    control-plane,master   7d18h   v1.21.0   10.10.13.48   <none>        CentOS Linux 7 (Core)   3.10.0-1160.21.1.el7.x86_64   docker://20.10.6
node-44     Ready    <none>                 7d18h   v1.21.0   10.10.13.44   <none>        CentOS Linux 7 (Core)   3.10.0-1160.21.1.el7.x86_64   docker://20.10.6
node-49     Ready    <none>                 7d18h   v1.21.0   10.10.13.49   <none>        CentOS Linux 7 (Core)   3.10.0-1160.21.1.el7.x86_64   docker://20.10.6

[root@master-45 ~]# kubectl get no -owide
NAME        STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
master-45   Ready    control-plane,master   18h   v1.21.0   10.10.13.45   <none>        CentOS Linux 7 (Core)   3.10.0-1160.21.1.el7.x86_64   docker://20.10.6
node-43     Ready    <none>                 18h   v1.21.0   10.10.13.43   <none>        CentOS Linux 7 (Core)   3.10.0-1160.21.1.el7.x86_64   docker://20.10.6
node-46     Ready    <none>                 18h   v1.21.0   10.10.13.46   <none>        CentOS Linux 7 (Core)   3.10.0-1160.21.1.el7.x86_64   docker://20.10.6
```

master-48 所在集群对应为cluster1集群，master-45 所在集群对应为cluster2集群。


2.istio-1.11.5
下载地址：[istio-1.11.5-linux-amd64.tar.gz](https://github.com/istio/istio/releases/download/1.11.5/istio-1.11.5-linux-amd64.tar.gz)

3.为两个集群生成中间证书

多集群服务网格部署要求你在网格中的所有集群之间建立信任关系，需要使用一个公共根，为每个集群生成中间证书。

默认情况下，Istio CA 会生成一个自签名的根证书和密钥，并使用它们来签署工作负载证书。为了保护根 CA 密钥，您应该使用在安全机器上离线运行的根 CA，并使用根 CA 向运行在每个集群上的 Istio CA 签发中间证书。Istio CA 可以使用管理员指定的证书和密钥来签署工作负载证书，并将管理员指定的根证书作为信任根分配给工作负载。

下图展示了在包含两个集群的网格中推荐的 CA 层次结构。

![istio-multiple-ca](../images/istio-multiple-ca.png)

在 Istio 安装包的顶层目录下，创建一个目录来存放证书和密钥：
```shell
$ mkdir -p certs
$ pushd certs
```
生成根证书和密钥：
```shell
$ make -f ../tools/certs/Makefile.selfsigned.mk root-ca
```

对于每个集群，为 Istio CA 生成一个中间证书和密钥。
```shell
$ make -f ../tools/certs/Makefile.selfsigned.mk cluster1-cacerts
$ make -f ../tools/certs/Makefile.selfsigned.mk cluster2-cacerts
```

生成后的效果：
```shell
[root@master-45 ~]# cd istio-1.11.5/
[root@master-45 istio-1.11.5]# ls
bin  certs  LICENSE  manifests  manifest.yaml  README.md  samples  tools
[root@master-45 istio-1.11.5]# cd certs/
[root@master-45 certs]# ls
cluster1  cluster2  root-ca.conf  root-cert.csr  root-cert.pem  root-cert.srl  root-key.pem
[root@master-45 certs]# ls cluster1/
ca-cert.pem  ca-key.pem  cert-chain.pem  root-cert.pem
[root@master-45 certs]#
```
将 istio-1.11.5 scp到 cluster2中。

在 cluster1 集群上：
```shell
$ kubectl create namespace istio-system
$ kubectl create secret generic cacerts -n istio-system \
      --from-file=cluster1/ca-cert.pem \
      --from-file=cluster1/ca-key.pem \
      --from-file=cluster1/root-cert.pem \
      --from-file=cluster1/cert-chain.pem
```

在 cluster2 集群上：
```shell
$ kubectl create namespace istio-system
$ kubectl create secret generic cacerts -n istio-system \
      --from-file=cluster2/ca-cert.pem \
      --from-file=cluster2/ca-key.pem \
      --from-file=cluster2/root-cert.pem \
      --from-file=cluster2/cert-chain.pem
```

返回 Istio 安装的顶层目录：
```shell
$ popd
```

### 跨网络多主架构的安装

在 cluster1 和 cluster2 两个集群上，安装 Istio 控制平面， 且将两者均设置为主集群（primary cluster）。 集群 cluster1 在 network1 网络上，而集群 cluster2 在 network2 网络上。 这意味着这些跨集群边界的 Pod 之间，网络不能直接连通。

跨集群边界的服务负载，通过专用的东西向网关，以间接的方式通讯。每个集群中的网关在其他集群必须可以访问。

![multiple-primary-cluster-on-separate-networks](../images/istio-multiple-multiple-primary-cluster-on-separate-networks.png)

说明
> cluster1在network1网络，cluster2在network2网络，cluster1和cluster2有各自istiod。
> 
> cluster1的istiod监控cluster1和cluster2的apiserver，cluster2的istiod监控cluster1和cluster2的apiserver。
> 
> cluster1的service连接到cluster1的istiod，cluster2的service连接到cluster2的istiod。
> cluster1的service通过cluster2的东西向网关访问cluster2的service，cluster2的service通过cluster1的东西向网关访问cluster1的service。

1.给istio-system namespace打标签

```shell
# cluster1
kubectl  label namespace istio-system topology.istio.io/network=network1

# cluster2
kubectl  label namespace istio-system topology.istio.io/network=network2
```

2.生成istio operator部署文件
```shell
# cluster1
cat <<EOF > cluster1.yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  values:
    global:
      meshID: mesh1
      multiCluster:
        clusterName: cluster1
      network: network1
EOF

# cluster2
cat <<EOF > cluster2.yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  values:
    global:
      meshID: mesh1
      multiCluster:
        clusterName: cluster2
      network: network2
EOF
```

3.安装一个提供 集群 API Server 访问权限的远程 Secret

````shell
# 在 cluster1 中安装一个提供 cluster2 API Server 访问权限的远程 Secret。
## 首先在48节点生成cluster1的remote Secret，然后scp给cluster2。然后再apply 45 scp过来的cluster2的remote secret
istioctl x create-remote-secret --name=cluster1  --server=https://10.10.13.48:6443 > remote-secret-cluster1.yaml
scp remote-secret-cluster1.yaml root@10.10.13.45:/root

kubectl apply -f remote-secret-cluster2.yaml

# 在 cluster2 中安装一个提供 cluster1 API Server 访问权限的远程 Secret。
# 同上，也是先生成自己集群的remote secret ，然后apply cluster1的remote secret。
istioctl x create-remote-secret --name=cluster2  --server=https://10.10.13.45:6443 > remote-secret-cluster2.yaml
scp remote-secret-cluster2.yaml root@10.10.13.48:/root

kubectl apply -f remote-secret-cluster1.yaml
````

这样，cluster1集群就可以监听cluster2的apiServer，同理，cluster2集群也可以监听cluster1的apiServer.

4.部署istio
```shell
# 在cluster1
istioctl install -f cluster1.yaml

# 在cluster2
istioctl install -f cluster2.yaml
```

5.部署东西向网关

```shell
# 在cluster1
## 部署部署东西向网关
/root/istio-1.11.5/samples/multicluster/gen-eastwest-gateway.sh --mesh mesh1 --cluster cluster1 --network network1 | istioctl  install -y  -f -

## 配置东西向网关ip ,这里我设置为cluster1的 master ip
kubectl patch svc  -n istio-system istio-eastwestgateway -p '{"spec":{"externalIPs":["10.10.13.48"]}}'


# 在cluster2
## 部署部署东西向网关
/root/istio-1.11.5/samples/multicluster/gen-eastwest-gateway.sh --mesh mesh1 --cluster cluster2 --network network2 | istioctl  install -y  -f -

## 配置东西向网关ip ,这里我设置为cluster2的 master ip
kubectl patch svc  -n istio-system istio-eastwestgateway -p '{"spec":{"externalIPs":["10.10.13.45"]}}'
```

6.暴露服务
```shell
# cluster1 和 cluster2
kubectl  apply -n istio-system -f /root/istio-1.11.5/samples/multicluster/expose-services.yaml
```

7.重启
```shell
# cluster1 和 cluster2
kubectl rollout restart deploy -n istio-system
```

8.部署业务测试容器
```shell
# 设置自动注入(cluster1和cluster2)
kubectl label namespace default istio-injection=enabled

# 部署bookinfo(cluster1和cluster2)
kubectl apply -f  istio-1.11.5/samples/bookinfo/platform/kube/bookinfo.yaml

# 部署helloworld
## cluster1
kubectl apply -f istio-1.11.5/samples/helloworld/helloworld.yaml -l service=helloworld
kubectl apply -f istio-1.11.5/samples/helloworld/helloworld.yaml -l version=v1

## cluster2
kubectl apply -f istio-1.11.5/samples/helloworld/helloworld.yaml -l service=helloworld
kubectl apply -f istio-1.11.5/samples/helloworld/helloworld.yaml -l version=v2

# 部署sleep(cluster1和cluster2)
kubectl apply -f  istio-1.11.5/samples/sleep/sleep.yaml

```

9.查看po效果
```shell
# cluster1
$ kubectl get po -A
NAMESPACE      NAME                                     READY   STATUS    RESTARTS   AGE
default        details-v1-79f774bdb9-r5kw4              2/2     Running   0          90m
default        helloworld-v1-776f57d5f6-8sb7f           2/2     Running   0          78m
default        productpage-v1-6b746f74dc-2qhqx          2/2     Running   0          90m
default        ratings-v1-b6994bb9-88mw2                2/2     Running   0          90m
default        reviews-v1-545db77b95-df5cz              2/2     Running   0          90m
default        reviews-v2-7bf8c9648f-6ckfp              2/2     Running   0          90m
default        reviews-v3-84779c7bbc-s2rkl              2/2     Running   0          90m
default        sleep-557747455f-7h9hd                   2/2     Running   0          82m
istio-system   istio-eastwestgateway-68786fb975-jhhpl   1/1     Running   0          89m
istio-system   istio-ingressgateway-5cb96858b5-4zthw    1/1     Running   0          89m
istio-system   istiod-64dfdcc9db-j7plx                  1/1     Running   0          89m
kube-system    coredns-558bd4d5db-j9ndm                 1/1     Running   0          7d19h
kube-system    coredns-558bd4d5db-pcbfv                 1/1     Running   0          7d19h
kube-system    etcd-master-48                           1/1     Running   0          7d19h
kube-system    kube-apiserver-master-48                 1/1     Running   0          7d19h
kube-system    kube-controller-manager-master-48        1/1     Running   0          7d19h
kube-system    kube-proxy-2p95b                         1/1     Running   0          7d19h
kube-system    kube-proxy-ctrnj                         1/1     Running   0          7d19h
kube-system    kube-proxy-smkdv                         1/1     Running   0          7d19h
kube-system    kube-scheduler-master-48                 1/1     Running   0          7d19h
kube-system    weave-net-42s84                          2/2     Running   0          7d19h
kube-system    weave-net-k6gzx                          2/2     Running   0          7d19h
kube-system    weave-net-ks9xt                          2/2     Running   1          7d19h

# cluster2
$ kubectl get po -A
NAMESPACE      NAME                                     READY   STATUS    RESTARTS   AGE
default        details-v1-79f774bdb9-9cnwk              2/2     Running   0          90m
default        helloworld-v2-54df5f84b-mk9zj            2/2     Running   0          77m
default        productpage-v1-6b746f74dc-cqwd7          2/2     Running   0          90m
default        ratings-v1-b6994bb9-lf7zx                2/2     Running   0          90m
default        reviews-v1-545db77b95-vcgl6              2/2     Running   0          90m
default        reviews-v2-7bf8c9648f-nq6zz              2/2     Running   0          90m
default        reviews-v3-84779c7bbc-zptjb              2/2     Running   0          90m
default        sleep-557747455f-5hcsm                   2/2     Running   0          84m
istio-system   istio-eastwestgateway-86f64d89b7-zmrln   1/1     Running   0          90m
istio-system   istio-ingressgateway-f5648b4b9-l6jbs     1/1     Running   0          90m
istio-system   istiod-7cb788d479-vfz5x                  1/1     Running   0          90m
kube-system    coredns-558bd4d5db-wwhc8                 1/1     Running   0          19h
kube-system    coredns-558bd4d5db-xkhsj                 1/1     Running   0          19h
kube-system    etcd-master-45                           1/1     Running   0          19h
kube-system    kube-apiserver-master-45                 1/1     Running   0          19h
kube-system    kube-controller-manager-master-45        1/1     Running   0          19h
kube-system    kube-proxy-m49vz                         1/1     Running   0          19h
kube-system    kube-proxy-qfq98                         1/1     Running   0          19h
kube-system    kube-proxy-xmxns                         1/1     Running   0          19h
kube-system    kube-scheduler-master-45                 1/1     Running   0          19h
kube-system    weave-net-2x5nl                          2/2     Running   0          19h
kube-system    weave-net-5thfp                          2/2     Running   0          19h
kube-system    weave-net-wdpwm                          2/2     Running   1          19h
```

验证：

cluster1:
```shell
$ istioctl ps
NAME                                                    CDS        LDS        EDS        RDS          ISTIOD                      VERSION
details-v1-79f774bdb9-r5kw4.default                     SYNCED     SYNCED     SYNCED     SYNCED       istiod-64dfdcc9db-j7plx     1.11.5
helloworld-v1-776f57d5f6-8sb7f.default                  SYNCED     SYNCED     SYNCED     SYNCED       istiod-64dfdcc9db-j7plx     1.11.5
istio-eastwestgateway-68786fb975-jhhpl.istio-system     SYNCED     SYNCED     SYNCED     NOT SENT     istiod-64dfdcc9db-j7plx     1.11.5
istio-ingressgateway-5cb96858b5-4zthw.istio-system      SYNCED     SYNCED     SYNCED     NOT SENT     istiod-64dfdcc9db-j7plx     1.11.5
productpage-v1-6b746f74dc-2qhqx.default                 SYNCED     SYNCED     SYNCED     SYNCED       istiod-64dfdcc9db-j7plx     1.11.5
ratings-v1-b6994bb9-88mw2.default                       SYNCED     SYNCED     SYNCED     SYNCED       istiod-64dfdcc9db-j7plx     1.11.5
reviews-v1-545db77b95-df5cz.default                     SYNCED     SYNCED     SYNCED     SYNCED       istiod-64dfdcc9db-j7plx     1.11.5
reviews-v2-7bf8c9648f-6ckfp.default                     SYNCED     SYNCED     SYNCED     SYNCED       istiod-64dfdcc9db-j7plx     1.11.5
reviews-v3-84779c7bbc-s2rkl.default                     SYNCED     SYNCED     SYNCED     SYNCED       istiod-64dfdcc9db-j7plx     1.11.5
sleep-557747455f-7h9hd.default                          SYNCED     SYNCED     SYNCED     SYNCED       istiod-64dfdcc9db-j7plx     1.11.5

$ istioctl pc endpoint -n istio-system istio-ingressgateway-5cb96858b5-4zthw|grep productpage
10.10.13.45:15443                HEALTHY     OK                outbound|9080||productpage.default.svc.cluster.local
10.44.0.8:9080                   HEALTHY     OK                outbound|9080||productpage.default.svc.cluster.local
```

cluster2:
```shell
$ istioctl ps
NAME                                                    CDS        LDS        EDS        RDS          ISTIOD                      VERSION
details-v1-79f774bdb9-9cnwk.default                     SYNCED     SYNCED     SYNCED     SYNCED       istiod-7cb788d479-vfz5x     1.11.5
helloworld-v2-54df5f84b-mk9zj.default                   SYNCED     SYNCED     SYNCED     SYNCED       istiod-7cb788d479-vfz5x     1.11.5
istio-eastwestgateway-86f64d89b7-zmrln.istio-system     SYNCED     SYNCED     SYNCED     NOT SENT     istiod-7cb788d479-vfz5x     1.11.5
istio-ingressgateway-f5648b4b9-l6jbs.istio-system       SYNCED     SYNCED     SYNCED     NOT SENT     istiod-7cb788d479-vfz5x     1.11.5
productpage-v1-6b746f74dc-cqwd7.default                 SYNCED     SYNCED     SYNCED     SYNCED       istiod-7cb788d479-vfz5x     1.11.5
ratings-v1-b6994bb9-lf7zx.default                       SYNCED     SYNCED     SYNCED     SYNCED       istiod-7cb788d479-vfz5x     1.11.5
reviews-v1-545db77b95-vcgl6.default                     SYNCED     SYNCED     SYNCED     SYNCED       istiod-7cb788d479-vfz5x     1.11.5
reviews-v2-7bf8c9648f-nq6zz.default                     SYNCED     SYNCED     SYNCED     SYNCED       istiod-7cb788d479-vfz5x     1.11.5
reviews-v3-84779c7bbc-zptjb.default                     SYNCED     SYNCED     SYNCED     SYNCED       istiod-7cb788d479-vfz5x     1.11.5
sleep-557747455f-5hcsm.default                          SYNCED     SYNCED     SYNCED     SYNCED       istiod-7cb788d479-vfz5x     1.11.5

$ istioctl pc endpoint -n istio-system istio-ingressgateway-f5648b4b9-l6jbs.istio-system|grep productpage
10.10.13.48:15443                HEALTHY     OK                outbound|9080||productpage.default.svc.cluster.local
10.36.0.7:9080                   HEALTHY     OK                outbound|9080||productpage.default.svc.cluster.local
```

测试跨集群流量

在cluster1的sleep 容器中访问 helloworld.default 这个svc，由于cluster1中只部署了helloworld v1版本，而cluster2中只部署了helloworld v2版本。所以，如果响应是v1、v2切换，那么则表示我们的多集群部署成功了。

```shell
$ kubectl exec -it sleep-557747455f-7h9hd sh                                                            
$ kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ $ for i in `seq 20`;do curl -sS helloworld.default:5000/hello;done
Hello version: v1, instance: helloworld-v1-776f57d5f6-8sb7f
Hello version: v2, instance: helloworld-v2-54df5f84b-mk9zj
Hello version: v2, instance: helloworld-v2-54df5f84b-mk9zj
Hello version: v1, instance: helloworld-v1-776f57d5f6-8sb7f
Hello version: v2, instance: helloworld-v2-54df5f84b-mk9zj
Hello version: v1, instance: helloworld-v1-776f57d5f6-8sb7f
Hello version: v1, instance: helloworld-v1-776f57d5f6-8sb7f
Hello version: v2, instance: helloworld-v2-54df5f84b-mk9zj
Hello version: v2, instance: helloworld-v2-54df5f84b-mk9zj
Hello version: v1, instance: helloworld-v1-776f57d5f6-8sb7f
Hello version: v1, instance: helloworld-v1-776f57d5f6-8sb7f
Hello version: v2, instance: helloworld-v2-54df5f84b-mk9zj
Hello version: v1, instance: helloworld-v1-776f57d5f6-8sb7f
Hello version: v2, instance: helloworld-v2-54df5f84b-mk9zj
Hello version: v2, instance: helloworld-v2-54df5f84b-mk9zj
Hello version: v1, instance: helloworld-v1-776f57d5f6-8sb7f
Hello version: v1, instance: helloworld-v1-776f57d5f6-8sb7f
Hello version: v2, instance: helloworld-v2-54df5f84b-mk9zj
Hello version: v2, instance: helloworld-v2-54df5f84b-mk9zj
Hello version: v1, instance: helloworld-v1-776f57d5f6-8sb7f
/ $
```

结果很给力，多集群部署成功！



## 卸载
```shell
$ istioctl x uninstall --purge
$ kubectl delete namespace istio-system
```


















