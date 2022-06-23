# metallb
MetalLB 是一个负载均衡器，专门解决裸金属 Kubernetes 集群中无法使用 LoadBalancer 类型服务的痛点。MetalLB 使用标准化的路由协议，以便裸金属 Kubernetes 集群上的外部服务也尽可能地工作。即 MetalLB 能够帮助你在裸金属 Kubernetes 集群中创建 LoadBalancer 类型的 Kubernetes 服务。

在云环境中，当你请求一个负载均衡器时，云平台会自动分配一个负载均衡器的 IP 地址给你，应用程序通过此 IP 来访问经过负载均衡处理的服务。

使用 MetalLB 时，MetalLB 会自己为用户的 LoadBalancer 类型 Service 分配 IP 地址，当然该 IP 地址不是凭空产生的，需要用户在配置中提供一个 IP 地址池，Metallb 将会在其中选取地址分配给服务。

## 安装

```shell
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
```

资源详情：

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: metallb-system
  labels:
    app: metallb
---

apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  labels:
    app: metallb
  name: controller
spec:
  allowPrivilegeEscalation: false
  allowedCapabilities: []
  allowedHostPaths: []
  defaultAddCapabilities: []
  defaultAllowPrivilegeEscalation: false
  fsGroup:
    ranges:
      - max: 65535
        min: 1
    rule: MustRunAs
  hostIPC: false
  hostNetwork: false
  hostPID: false
  privileged: false
  readOnlyRootFilesystem: true
  requiredDropCapabilities:
    - ALL
  runAsUser:
    ranges:
      - max: 65535
        min: 1
    rule: MustRunAs
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    ranges:
      - max: 65535
        min: 1
    rule: MustRunAs
  volumes:
    - configMap
    - secret
    - emptyDir
---
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  labels:
    app: metallb
  name: speaker
spec:
  allowPrivilegeEscalation: false
  allowedCapabilities:
    - NET_RAW
  allowedHostPaths: []
  defaultAddCapabilities: []
  defaultAllowPrivilegeEscalation: false
  fsGroup:
    rule: RunAsAny
  hostIPC: false
  hostNetwork: true
  hostPID: false
  hostPorts:
    - max: 7472
      min: 7472
    - max: 7946
      min: 7946
  privileged: true
  readOnlyRootFilesystem: true
  requiredDropCapabilities:
    - ALL
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
    - configMap
    - secret
    - emptyDir
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: metallb
  name: controller
  namespace: metallb-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: metallb
  name: speaker
  namespace: metallb-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app: metallb
  name: metallb-system:controller
rules:
  - apiGroups:
      - ''
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - services/status
    verbs:
      - update
  - apiGroups:
      - ''
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - policy
    resourceNames:
      - controller
    resources:
      - podsecuritypolicies
    verbs:
      - use
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app: metallb
  name: metallb-system:speaker
rules:
  - apiGroups:
      - ''
    resources:
      - services
      - endpoints
      - nodes
    verbs:
      - get
      - list
      - watch
  - apiGroups: ["discovery.k8s.io"]
    resources:
      - endpointslices
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - policy
    resourceNames:
      - speaker
    resources:
      - podsecuritypolicies
    verbs:
      - use
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    app: metallb
  name: config-watcher
  namespace: metallb-system
rules:
  - apiGroups:
      - ''
    resources:
      - configmaps
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    app: metallb
  name: pod-lister
  namespace: metallb-system
rules:
  - apiGroups:
      - ''
    resources:
      - pods
    verbs:
      - list
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    app: metallb
  name: controller
  namespace: metallb-system
rules:
  - apiGroups:
      - ''
    resources:
      - secrets
    verbs:
      - create
  - apiGroups:
      - ''
    resources:
      - secrets
    resourceNames:
      - memberlist
    verbs:
      - list
  - apiGroups:
      - apps
    resources:
      - deployments
    resourceNames:
      - controller
    verbs:
      - get
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    app: metallb
  name: metallb-system:controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: metallb-system:controller
subjects:
  - kind: ServiceAccount
    name: controller
    namespace: metallb-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    app: metallb
  name: metallb-system:speaker
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: metallb-system:speaker
subjects:
  - kind: ServiceAccount
    name: speaker
    namespace: metallb-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    app: metallb
  name: config-watcher
  namespace: metallb-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: config-watcher
subjects:
  - kind: ServiceAccount
    name: controller
  - kind: ServiceAccount
    name: speaker
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    app: metallb
  name: pod-lister
  namespace: metallb-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-lister
subjects:
  - kind: ServiceAccount
    name: speaker
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    app: metallb
  name: controller
  namespace: metallb-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: controller
subjects:
  - kind: ServiceAccount
    name: controller
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: metallb
    component: speaker
  name: speaker
  namespace: metallb-system
spec:
  selector:
    matchLabels:
      app: metallb
      component: speaker
  template:
    metadata:
      annotations:
        prometheus.io/port: '7472'
        prometheus.io/scrape: 'true'
      labels:
        app: metallb
        component: speaker
    spec:
      containers:
        - args:
            - --port=7472
            - --config=config
            - --log-level=info
          env:
            - name: METALLB_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: METALLB_HOST
              valueFrom:
                fieldRef:
                  fieldPath: status.hostIP
            - name: METALLB_ML_BIND_ADDR
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            # needed when another software is also using memberlist / port 7946
            # when changing this default you also need to update the container ports definition
            # and the PodSecurityPolicy hostPorts definition
            #- name: METALLB_ML_BIND_PORT
            #  value: "7946"
            - name: METALLB_ML_LABELS
              value: "app=metallb,component=speaker"
            - name: METALLB_ML_SECRET_KEY
              valueFrom:
                secretKeyRef:
                  name: memberlist
                  key: secretkey
          image: quay.io/metallb/speaker:v0.12.0
          name: speaker
          ports:
            - containerPort: 7472
              name: monitoring
            - containerPort: 7946
              name: memberlist-tcp
            - containerPort: 7946
              name: memberlist-udp
              protocol: UDP
          livenessProbe:
            httpGet:
              path: /metrics
              port: monitoring
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 1
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /metrics
              port: monitoring
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 1
            successThreshold: 1
            failureThreshold: 3
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              add:
                - NET_RAW
              drop:
                - ALL
            readOnlyRootFilesystem: true
      hostNetwork: true
      nodeSelector:
        kubernetes.io/os: linux
      serviceAccountName: speaker
      terminationGracePeriodSeconds: 2
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/master
          operator: Exists
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: metallb
    component: controller
  name: controller
  namespace: metallb-system
spec:
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: metallb
      component: controller
  template:
    metadata:
      annotations:
        prometheus.io/port: '7472'
        prometheus.io/scrape: 'true'
      labels:
        app: metallb
        component: controller
    spec:
      containers:
        - args:
            - --port=7472
            - --config=config
            - --log-level=info
          env:
            - name: METALLB_ML_SECRET_NAME
              value: memberlist
            - name: METALLB_DEPLOYMENT
              value: controller
          image: quay.io/metallb/controller:v0.12.0
          name: controller
          ports:
            - containerPort: 7472
              name: monitoring
          livenessProbe:
            httpGet:
              path: /metrics
              port: monitoring
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 1
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /metrics
              port: monitoring
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 1
            successThreshold: 1
            failureThreshold: 3
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - all
            readOnlyRootFilesystem: true
      nodeSelector:
        kubernetes.io/os: linux
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        fsGroup: 65534
      serviceAccountName: controller
      terminationGracePeriodSeconds: 0
```

## 配置Ip池
apply下面内容：
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: my-ip-space
      protocol: layer2
      addresses:
      - 10.10.13.41-10.10.13.42
```
10.10.13.42 跟我k8s集群同一个网段，且该ip未被实际使用。这里设置的10.10.13.42是一个虚拟ip，当访问10.10.13.42的时候，其实就是访问你的k8s集群。

所以在设置addresses的时候，要确认这些地址与当前k8s集群是同一个网段，且这些ip并未被其他的虚拟机使用。设置后，metallb会自动生效你配置的虚拟ip，当你设置service为LoadBalancer类型后,就会自动使用这些ip。

如果你想看到详细配置更新过程，可以使用以下类似命令查看。
```shell
kubectl logs -l component=speaker -n metallb-system
```

## 测试

接下来，我们来创建一个服务类型为 LoadBalancer 的 Nginx 服务来验证下。
```shell
$ cat nginx-test.yaml

apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1
        ports:
        - name: http
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: LoadBalancer
```

服务创建完成，运行 kubectl apply -f nginx-test.yaml 命令后，我们可以看到对应服务信息。

```shell
$ kubectl get svc nginx 
NAME    TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx   LoadBalancer   10.21.216.179   10.10.13.42   80:32094/TCP   5m37s
```

从输出结果，我们可以看到 LoadBalancer 类型的服务，并且分配的外部 IP 地址是地址池中的第一个 IP 10.10.13.42。

最后，我们通过 curl http://10.10.13.42 命令来验证下，发现可以正常显示 Nginx 的欢迎信息。

```shell
➜  ~ curl http://10.10.13.42
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Reference
[Kubernetes 私有集群负载均衡器终极解决方案 MetalLB](https://cloud.tencent.com/developer/article/1631910)

