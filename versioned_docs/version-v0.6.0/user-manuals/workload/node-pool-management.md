---
title: Node Pool Management
---

### 

### 1）安装Yurt-App-Manager组件

```shell
$ cd  yurt-app-manager
$ kubectl apply -f config/setup/all_in_one.yaml
```

等待Yurt-App-Manager组件安装成功，验证

```shell
$ kubectl get pod -n kube-system |grep yurt-app-manager
```



### 2）节点池使用Example

- 创建一个节点池

```shell
$ cat <<EOF | kubectl apply -f -
apiVersion: apps.openyurt.io/v1alpha1
kind: NodePool
metadata:
  name: beijing
spec:
  type: Cloud
EOF

$ cat <<EOF | kubectl apply -f -
apiVersion: apps.openyurt.io/v1alpha1
kind: NodePool
metadata:
  name: hangzhou
spec:
  type: Edge
  annotations:
    apps.openyurt.io/example: test-hangzhou
  labels:
    apps.openyurt.io/example: test-hangzhou
  taints:
  - key: apps.openyurt.io/example
    value: test-hangzhou
    effect: NoSchedule
EOF
```

- 使用kubectl get节点池信息

```shell
$ kubectl get np 

NAME       TYPE   READYNODES   NOTREADYNODES   AGE
beijing    Cloud                               35s
hangzhou   Edge                                28s
```

- 将节点加入到节点池

添加云端节点Cloud node到北京节点池，你只需将此节点按如下方式打上label即可

```shell
$ kubectl label node {Your_Node_Name} apps.openyurt.io/desired-nodepool=beijing
```



```shell
For example:
$ kubectl label node master apps.openyurt.io/desired-nodepool=beijing

master labeled
```

当然，你也可以将你的边缘节点Edge node添加到杭州节点池，方法和上面类似

```shell
$ kubectl label node {Your_Node_Name} apps.openyurt.io/desired-nodepool=hangzhou
For example:
$ kubectl label node k8s-node1 apps.openyurt.io/desired-nodepool=hangzhou

k8s-node1 labeled

$ kubectl label node k8s-node2 apps.openyurt.io/desired-nodepool=hangzhou

k8s-node2 labeled
```

- 验证节点已经加入节点池

当Edge node成功加入到节点池，节点的配置信息除了节点池Spec中的所有内容，同时，节点添加了一个新的标签：apps.openyurt.io/nodepool。

```shell
$ kubectl get node {Your_Node_Name} -o yaml 

For Example:
$ kubectl get node k8s-node1 -o yaml

apiVersion: v1
kind: Node
metadata:
  annotations:
    apps.openyurt.io/example: test-hangzhou
    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
    node.alpha.kubernetes.io/ttl: "0"
    node.beta.alibabacloud.com/autonomy: "true"
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
  creationTimestamp: "2021-04-14T12:17:39Z"
  labels:
    apps.openyurt.io/desired-nodepool: hangzhou
    apps.openyurt.io/example: test-hangzhou
    apps.openyurt.io/nodepool: hangzhou
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: k8s-node1
    kubernetes.io/os: linux
    openyurt.io/is-edge-worker: "true"
  name: k8s-node1
  resourceVersion: "1244431"
  selfLink: /api/v1/nodes/k8s-node1
  uid: 1323f90b-acf3-4443-a7dc-7a54c212506c
spec:
  podCIDR: 192.168.1.0/24
  podCIDRs:
  - 192.168.1.0/24
  taints:
  - effect: NoSchedule
    key: apps.openyurt.io/example
    value: test-hangzhou
status:
***
```

