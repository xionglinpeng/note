# Pod



## 镜像拉取策略

默认情况下，部署Pod的时候，无论其对应Pod的容器镜像是否存在，都会重新拉取。但这是不必要的，我们常常希望如果镜像在节点存在，就直接使用。幸运的是，Pod资源可以设置其镜像的拉取策略。

Pod资源通过`spec.containers.imagePullPolicy`来设置其镜像的拉取策略，可选值如下：



下面是一个pod示例，通过`imagePullPolicy`来设置镜像的拉取策略：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
spec:
  containers:
  - name: busybox
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c","sleep 864000"]
```

## 调度Pod到指定节点

在某些情况下，需要将Pod调度到指定节点，例如使用了`hostPath`类型的volume。可以通过设置节点标签以决定需要调度到哪些节点。

下面是一个pod示例，通过`nodeSelector`来设置被调度的节点：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
spec:
  nodeSelector:
    app: work
  containers:
  - name: busybox
    image: busybox:latest
    command: ["/bin/sh","-c","sleep 864000"]
```

`app`为节点标签的key，`work`为节点标签对应key的值。

**查看节点label**

```shell
$ kubectl get node --show-labels
NAME                    STATUS   ROLES    AGE   VERSION   LABELS
localhost.localdomain   Ready    master   45d   v1.18.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=localhost.localdomain,kubernetes.io/os=linux,node-role.kubernetes.io/master=
work1                   Ready    <none>   45d   v1.18.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=work1,kubernetes.io/os=linux
work2                   Ready    <none>   37d   v1.18.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=work2,kubernetes.io/os=linux
```

**创建节点label**

语法：`kubectl label node <node-name> <key>=<value>`

示例：

```shell
$ kubectl label node work1 app=dev
node/work1 labeled
```

**删除节点label**

语法：`kubectl label node <node-name> <key>-`

示例：

```shell
$ kubectl label node work1 app-
node/work1 labeled
```

