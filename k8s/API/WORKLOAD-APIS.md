# WORKLOAD-APIS



## Container V1 Core





| Field             | Type           | Description                                                  |
| ----------------- | -------------- | ------------------------------------------------------------ |
| `args`            | *string array* |                                                              |
| `command`         | *string array* |                                                              |
| `env`             | *array*        | 要在容器中设置的环境变量列表，不能被更新。                   |
| `image`           | *string*       | docker镜像名称                                               |
| `imagePullPolicy` | *string*       | 镜像拉取策略，可选值`Always`、`Never`、`IfNotPresent`，如果镜像指定了`:latest`标签，则默认为`Always`，否则为`IfNotPresent`。不能被更新。更多信息：https://kubernetes.io/docs/concepts/containers/images#updating-images |
| `name`            | *string*       |                                                              |
| `ports`           | *array*        |                                                              |
| `volumeMounts`    | *array*        | 要挂载到容器文件系统中的Pod卷，不能被更新。                  |
|                   |                |                                                              |

name

指定为DNS_LABEL的容器的名称。pod中的每个容器必须有唯一的名称(DNS_LABEL)。不能被更新。

ports

要从容器公开的端口列表。在这里公开端口可以向系统提供关于容器使用的网络连接的附加信息，但主要是提供信息。在这里没有指定端口并不会阻止该端口被公开。任何监听容器内默认“0.0.0.0”地址的端口都可以从网络访问。不能被更新。









## ContainerPort v1 core

| Field           | Type      | Description                                                  |
| --------------- | --------- | ------------------------------------------------------------ |
| `containerPort` | *integer* | 要在Pod的IP地址上暴露的端口号。这必须是一个有效的端口号，`0 < x < 65536`。 |
| `hostIP`        | *string*  |                                                              |
| `hostPort`      | *integer* |                                                              |
| `name`          | *string*  |                                                              |
| `protocol`      | *string*  | 端口的协议，必须是`UDP`，`TCP`或`SCTP`。默认是`TCP`。        |



hostIP

将外部端口绑定到哪个主机IP。

hostPort

要在主机上公开的端口数。如果指定，这必须是一个有效的端口号，0 < x < 65536。如果指定了主机网络，这必须匹配ContainerPort。大多数容器不需要这个。

name

如果指定，这必须是IANA_SVC_NAME，并且在pod中是惟一的。pod中的每个命名端口必须有唯一的名称。可由服务引用的端口的名称。











## VolumeMount v1 core



| Field              | Type      | Description                                                  |
| ------------------ | --------- | ------------------------------------------------------------ |
| `mountPath`        | *string*  |                                                              |
| `mountPropagation` | *string*  |                                                              |
| `name`             | *string*  |                                                              |
| `readOnly`         | *boolean* | 如果为`true`，则以只读的方式挂载，否则以读写的方式挂载（为`false`或者未指定）。默认为`false`； |
| `subPath`          | *string*  |                                                              |
| `subPathExpr`      | *string*  |                                                              |
|                    |           |                                                              |

mountPath

容器内挂载卷的路径，不能包含'.'。

mountPropagation

mountPropagation确定挂载如何从宿主传播到容器，以及如何从容器传播到宿主。未设置时，将使用MountPropagationNone。这是1。10的beta。

name

必须与卷的名称匹配

readOnly

如果为true，则以只读的方式挂载，否则以读写的方式挂载（为false或者未指定）。默认为false；

subPath

在容器的卷中安装的路径。默认值为“”(卷的根)。

subPathExpr

应从其中装载容器卷的卷内展开的路径。行为类似于subPath，但是环境变量引用$(VAR_NAME)是使用容器的环境展开的。默认值为“”(卷的根)。SubPathExpr和subPath是互斥的。



## Pod VI Core



affinity