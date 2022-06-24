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