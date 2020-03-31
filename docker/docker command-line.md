# Command-Line Interfaces

## Docker CLI

### docker image



```java
[root@localhost nacos-docker]# docker image

Usage:	docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Run 'docker image COMMAND --help' for more information on a command.
[root@localhost nacos-docker]# docker tag
"docker tag" requires exactly 2 arguments.
See 'docker tag --help'.

Usage:  docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
```



## run

`-d`

`-p``--publish`

```shell
-p <ip>:<host-port>:<container-port>
```



`-P`

`--name`

`--link`

```shell
$ docker run -d --link mysql:database
```

`-e`

`-v`

`--network`



## network



help

```shell
docker@docker0:/mnt/sda1/var/lib/home/apollo$ docker network --help                                                                                                              
Usage:	docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
```



- `connect`：
- `create`：
- `disconnect`：
- `inspect`：
- `ls`：
- `prune`：
- `rm`：



### create





```shell
$ docker network create -d overlay swarm-database-mysql
dgjokw6xxdn46ax6x706lz8b6
```





## inspect





## Management Commands

### swarm



*help*

```shell
docker@docker2:~$ docker swarm --help                                                                                                                                            

Usage:	docker swarm COMMAND

Manage Swarm

Commands:
  ca          Display and rotate the root CA
  init        Initialize a swarm
  join        Join a swarm as a node and/or manager
  join-token  Manage join tokens
  leave       Leave the swarm
  unlock      Unlock swarm
  unlock-key  Manage the unlock key
  update      Update the swarm

Run 'docker swarm COMMAND --help' for more information on a command.
```





#### leave

退出集群。

*help：*

```shell
docker@docker2:~$ docker swarm leave --help

Usage:	docker swarm leave [OPTIONS]

Leave the swarm

Options:
  -f, --force   Force this node to leave the swarm, ignoring warnings
```

*示例：*

```shell
$ docker swarm leave                                        
Node left the swarm.
```



### node



help

```shell
$ docker node --help                                                                                                                                             
Usage:	docker node COMMAND

Manage Swarm nodes

Commands:
  demote      Demote one or more nodes from manager in the swarm
  inspect     Display detailed information on one or more nodes
  ls          List nodes in the swarm
  promote     Promote one or more nodes to manager in the swarm
  ps          List tasks running on one or more nodes, defaults to current node
  rm          Remove one or more nodes from the swarm
  update      Update a node

Run 'docker node COMMAND --help' for more information on a command.
```



- `demote`：
- `inspect`：
- `ls`：列出swarm中的节点。
- `promote`：
- `ps`：
- `rm`：从swarm中删除一个或多个节点。
- `update`：更新节点。



#### demote

#### inspect

#### ls

#### promote

#### ps

#### rm

#### update

### service



help

```shell
docker@docker0:/mnt/sda1/var/lib/home/apollo$ docker service --help                                                                                                              
Usage:	docker service COMMAND

Manage services

Commands:
  create      Create a new service
  inspect     Display detailed information on one or more services
  logs        Fetch the logs of a service or task
  ls          List services
  ps          List the tasks of one or more services
  rm          Remove one or more services
  rollback    Revert changes to a service's configuration
  scale       Scale one or multiple replicated services
  update      Update a service

Run 'docker service COMMAND --help' for more information on a command.
```





- `create`：
- `inspect`：
- `logs`：
- `ls`：列出服务。
- `ps`：
- `rm`：删除一个或多个服务。
- `rollback`：
- `scale`：
- `update`：



#### create

help

```shell
docker@docker0:/mnt/sda1/var/lib/home/apollo$ docker service create -help                                                                                                        
Flag shorthand -h has been deprecated, please use --help

Usage:	docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]

Create a new service

Options:
      --config config                      Specify configurations to expose to the service
      --constraint list                    Placement constraints
      --container-label list               Container labels
      --credential-spec credential-spec    Credential spec for managed service account (Windows only)
  -d, --detach                             Exit immediately instead of waiting for the service to converge
      --dns list                           Set custom DNS servers
      --dns-option list                    Set DNS options
      --dns-search list                    Set custom DNS search domains
      --endpoint-mode string               Endpoint mode (vip or dnsrr) (default "vip")
      --entrypoint command                 Overwrite the default ENTRYPOINT of the image
  -e, --env list                           Set environment variables (default )
      --env-file list                      Read in a file of environment variables
      --generic-resource list              User defined resources
      --group list                         Set one or more supplementary user groups for the container
      --health-cmd string                  Command to run to check health
      --health-interval duration           Time between running the check (ms|s|m|h)
      --health-retries int                 Consecutive failures needed to report unhealthy
      --health-start-period duration       Start period for the container to initialize before counting retries towards unstable (ms|s|m|h)
      --health-timeout duration            Maximum time to allow one check to run (ms|s|m|h)
      --host list                          Set one or more custom host-to-IP mappings (host:ip)
      --hostname string                    Container hostname
      --init                               Use an init inside each service container to forward signals and reap processes
      --isolation string                   Service container isolation mode
  -l, --label list                         Service labels
      --limit-cpu decimal                  Limit CPUs
      --limit-memory bytes                 Limit Memory
      --log-driver string                  Logging driver for service
      --log-opt list                       Logging driver options
      --mode string                        Service mode (replicated or global) (default "replicated")
      --mount mount                        Attach a filesystem mount to the service
      --name string                        Service name
      --network network                    Network attachments
      --no-healthcheck                     Disable any container-specified HEALTHCHECK
      --no-resolve-image                   Do not query the registry to resolve image digest and supported platforms
      --placement-pref pref                Add a placement preference
  -p, --publish port                       Publish a port as a node port
  -q, --quiet                              Suppress progress output
      --read-only                          Mount the container's root filesystem as read only
      --replicas uint                      Number of tasks
      --replicas-max-per-node uint         Maximum number of tasks per node (default 0 = unlimited)
      --reserve-cpu decimal                Reserve CPUs
      --reserve-memory bytes               Reserve Memory
      --restart-condition string           Restart when condition is met ("none"|"on-failure"|"any") (default "any")
      --restart-delay duration             Delay between restart attempts (ns|us|ms|s|m|h) (default 5s)
      --restart-max-attempts uint          Maximum number of restarts before giving up
      --restart-window duration            Window used to evaluate the restart policy (ns|us|ms|s|m|h)
      --rollback-delay duration            Delay between task rollbacks (ns|us|ms|s|m|h) (default 0s)
      --rollback-failure-action string     Action on rollback failure ("pause"|"continue") (default "pause")
      --rollback-max-failure-ratio float   Failure rate to tolerate during a rollback (default 0)
      --rollback-monitor duration          Duration after each task rollback to monitor for failure (ns|us|ms|s|m|h) (default 5s)
      --rollback-order string              Rollback order ("start-first"|"stop-first") (default "stop-first")
      --rollback-parallelism uint          Maximum number of tasks rolled back simultaneously (0 to roll back all at once) (default 1)
      --secret secret                      Specify secrets to expose to the service
      --stop-grace-period duration         Time to wait before force killing a container (ns|us|ms|s|m|h) (default 10s)
      --stop-signal string                 Signal to stop the container
      --sysctl list                        Sysctl options
  -t, --tty                                Allocate a pseudo-TTY
      --update-delay duration              Delay between updates (ns|us|ms|s|m|h) (default 0s)
      --update-failure-action string       Action on update failure ("pause"|"continue"|"rollback") (default "pause")
      --update-max-failure-ratio float     Failure rate to tolerate during an update (default 0)
      --update-monitor duration            Duration after each task update to monitor for failure (ns|us|ms|s|m|h) (default 5s)
      --update-order string                Update order ("start-first"|"stop-first") (default "stop-first")
      --update-parallelism uint            Maximum number of tasks updated simultaneously (0 to update all at once) (default 1)
  -u, --user string                        Username or UID (format: <name|uid>[:<group|gid>])
      --with-registry-auth                 Send registry authentication details to swarm agents
  -w, --workdir string                     Working directory inside the container
```





```shell
$ docker service create --name mysql -p 3306:3306 --network swarm-database-mysql -e MYSQL_ROOT_PASSWORD=597646251 --replicas 1 mysql:8.0.19
```





#### logs

#### ls

#### ps

#### rm



### stack

*stack help*

```shell
docker@docker1:~$ docker stack --help                                                                                                                                            
Usage:	docker stack [OPTIONS] COMMAND

Manage Docker stacks

Options:
      --orchestrator string   Orchestrator to use (swarm|kubernetes|all)

Commands:
  deploy      Deploy a new stack or update an existing stack
  ls          List stacks
  ps          List the tasks in the stack
  rm          Remove one or more stacks
  services    List the services in the stack

Run 'docker stack COMMAND --help' for more information on a command.
```



- `deploy`：部署新stack，或更新现有stack。
- `ls`：列出stack。

- `ps`：列表stack中的任务。

- `rm`：删除一个或多个stack。

- `services`：列出stack中的服务。

  

#### deploy

*deploy help*

```shell
docker@docker1:~$ docker stack deploy --help                                                                                                                                     
Usage:	docker stack deploy [OPTIONS] STACK

Deploy a new stack or update an existing stack

Aliases:
  deploy, up

Options:
      --bundle-file string     Path to a Distributed Application Bundle file
  -c, --compose-file strings   Path to a Compose file, or "-" to read from stdin
      --orchestrator string    Orchestrator to use (swarm|kubernetes|all)
      --prune                  Prune services that are no longer referenced
      --resolve-image string   Query the registry to resolve image digest and supported platforms ("always"|"changed"|"never") (default "always")
      --with-registry-auth     Send registry authentication details to Swarm agents
```





- `--bundle-file`：
- `-c`, `--compose-file`：指向Compose文件的路径，或
- `--orchestrator`：
- `--prune`：删除不在被引用的服务。
- `--resolve-image`：
- `--with-registry-auth`：



#### ls



#### ps



#### rm



*help*

```shell
$ docker stack rm --help                                                                                                             
Usage:	docker stack rm [OPTIONS] STACK [STACK...]

Remove one or more stacks

Aliases:
  rm, remove, down

Options:
      --orchestrator string   Orchestrator to use (swarm|kubernetes|all)
```





#### services



*help*

```shell
$ docker stack services --help                                                                                                       
Usage:	docker stack services [OPTIONS] STACK

List the services in the stack

Options:
  -f, --filter filter         Filter output based on conditions provided
      --format string         Pretty-print services using a Go template
      --orchestrator string   Orchestrator to use (swarm|kubernetes|all)
  -q, --quiet                 Only display IDs
```





