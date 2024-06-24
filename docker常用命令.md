# Docker常用命令

## 容器生命周期管理

### docker run

创建一个新的容器并运行

- **-d:** 后台运行容器，并返回容器ID；

- **-i:** 以交互模式运行容器，通常与 -t 同时使用；

- **-t:** 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

- **--name="xxx":** 为容器指定一个名称；

- **-e username="ritchie":** 设置环境变量；

- **-m :**设置容器使用内存最大值；

- **--link=[]:** 添加链接到另一个容器；

- **--net="bridge":** 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；

- **--expose=[]:** 开放一个端口或一组端口；

- **-p:** 指定端口映射，格式为：主机(宿主)端口:容器端口(**-P**是随机端口映射)



### docker create

创建一个新的容器但**不启动它**（对比`docker run`）



### docker start

启动一个**已存在**且已停止的容器，而`docker run`是创建并运行一个**新的容器**



### docker stop

停止一个运行中的容器



### docker pause 

暂停容器中所有的进程。



### docker unpause 

恢复容器中所有的进程。



### docker exec

在运行的容器中执行命令



#### 示例1（打开正在后台运行的容器终端）

```shell
docker exec -it [容器ID或名称] /bin/bash
```

如果对应容器中没有`/bash`，可以换成`/sh`。



示例2

```shell
$ docker exec -it mynginx /bin/sh /root/runoob.sh
```

使用 `docker exec -it mynginx /bin/sh /root/runoob.sh`，会启动一个交互模式的 Shell (`/bin/sh`)，并在启动后立即执行 `runoob.sh` 脚本。



### `-i`和`-t`和`-d`

- 当需要向容器内的进程提供数据流时使用 `-i`，例如通过管道将数据传递给容器内的命令。
- 当需要在容器中进行交互式会话时使用 `-it`，例如通过 Bash 终端进行操作。
- 使用 `-d` 可以在后台运行命令，而不需要交互式终端。

#### 示例1（仅使用`-i`而不使用`-t`）

```shell
echo "Hello, Docker!" | docker exec -i mynginx /bin/sh -c 'cat > /root/message.txt'
```

在这个命令中：

- `echo "Hello, Docker!"` 生成一个输入流。
- `docker exec -i` 保持标准输入打开，使输入流可以传递给容器中的命令。
- `mynginx` 是容器的名称。
- `/bin/sh -c 'cat > /root/message.txt'` 在容器中执行的命令，把标准输入的内容写入 `/root/message.txt` 文件。

这个命令将主机上的 "Hello, Docker!" 内容通过标准输入传递给容器，并写入容器中的文件。



#### 示例2（使用`-d`进行后台执行命令）

```shell
docker exec -d mynginx /bin/sh -c "while true; do echo 'Hello, Docker!' >> /var/log/hello.log; sleep 5; done"
```

- `docker exec -d mynginx` 告诉 Docker 在 `mynginx` 容器中以分离模式（后台模式）运行命令。
- `/bin/sh -c "while true; do echo 'Hello, Docker!' >> /var/log/hello.log; sleep 5; done"` 是在容器中执行的一个典型的 Shell 命令，用于在 Unix/Linux 系统中执行循环操作，它会每5秒钟向 `/var/log/hello.log` 文件中追加一行 "Hello, Docker!"。

这个命令会在后台运行，而不会看到终端输出，但这个命令会持续在容器内运行并记录日志。



## 容器操作

### docker ps

显示正在运行的程序。如果想要展示所有容器（包括已停止的）加上选项`-a`。

`-a`: 显示所有的容器，包括未运行的。

`-p`: 静默模式，只显示容器id

`-s`: 显示总的文件大小。

```shell
rui@Rui:~/GitNote/Note$ docker ps -q -a
2982c1cf6281
rui@Rui:~/GitNote/Note$ docker ps -a -s
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES          SIZE
2982c1cf6281   hello-world   "/hello"   45 seconds ago   Exited (0) 45 seconds ago             goofy_colden   0B (virtual 1.84kB)
```

