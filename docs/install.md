## istio 的安装

### 安装

```shell
$ istioctl operator init
```
```shell
$ kubectl apply -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: example-istiocontrolplane
spec:
  profile: demo
EOF
```

### 卸载
```shell
$ kubectl delete istiooperators.install.istio.io -n istio-system example-istiocontrolplane

$ istioctl operator remove

$ istioctl manifest generate | kubectl delete -f -
$ kubectl delete ns istio-system --grace-period=0 --force
```

## zsh安装

[Centos7-Linux安装zsh和oh-my-zsh(内含国内安装方法)](https://blog.csdn.net/qimowei/article/details/119517167)

## 安装k8s
```shell
# 安装前设置Hostname
$ hostnamectl set-hostname master-48

$ ./k8sctl --master_ip 10.10.13.48 --node_ip 10.10.13.44 --node_ip 10.10.13.49 --user root --password password --installdocker --setup
linux

```

