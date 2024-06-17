# 手动构建docker镜像

## commit命令

```shell
docker commit 容器名称/ID 新的镜像名称
```

Docker官方并不推荐，使用者不知道镜像是如何构建出来的，也有安全性问题，并且构建效率低。

## Dockerfile

### 常用指令

#### FROM

格式为`FROM <image>`或者`FROM <image>:<tag>`
第一条指令必须为FROM指令。并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令(每个镜像一次)
示例：

```css
FROM centos:6.6
```

#### MAINTAINER

格式为`MAINTAINER <name>`，指定维护者信息
示例：

```dockerfile
MAINTAINER Breeze Yan<yan_ruo_gu@163.com>
```

#### RUN

格式为`RUN <command>`或者`RUN ["executable","param1","param2"]`
前者将在shell终端中运行命令，即/bin/sh -c；后者使用exec执行。每条RUN指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时，可以使用\来换行。
示例：

```css
RUN ["/bin/bash", "-c","echo hello"]
```

#### CMD

支持三种格式：

```dockerfile
CMD ["executable","param1","param2"]    #使用exec执行，推荐的方式
CMD command param1 param2   #在/bin/sh中执行，提供给需要交互的应用
CMD ["param1","param2"]    #提供给ENTRYPOINT的默认参数
```

指定启动窗口时执行的命令，每个Dockerfile只能有一条CMD命令。如果指定了多条，只有最后一条会被执行。如果用户启动容器时指定了运行的命令，则会覆盖掉CMD指定的命令
示例：

```css
CMD ["supervisord","-c","/etc/supervisord.conf"]
```

#### EXPOSE

格式为`EXPOSE <port> [<port>...]`
告诉docker容器暴露的端口，供互联系统使用。在启动容器时需要通过-P，docker主机会自动分配 一个端口转发到指定的端口，使用-p，则可以具体指定哪个本地端口映射过来。

示例：

```dockerfile
EXPOSE 22 80
```

#### ENV

格式为`ENV <key> <value>`
指定一个环境变量，会被后续RUN指令使用，并在容器运行时保持

示例：

```dockerfile
ENV TZ "Asia/Shanghai"
ENV TERM xterm
```

#### ADD

格式为`ADD <src> <dest>`
该命令将复制指定的`<src>`到容器中的`<dest>`。其中`<src>`可以是dockerfile所在目录的一个相对路径(文件或目录),也可以是一个URL，还可以是一个tar文件（自动解压为目录）

示例：

```dockerfile
ADD aliyun-mirror.repo /etc/yum.repos.d/CentOS-Base.repo
ADD aliyun-epel.repo /etc/yum.repos.d/epel.repo
```

#### COPY

格式为`COPY <src> <dest>`
与ADD类似，复制本地主机的`<src>`为容器中的`<dest>`，目标路径不存在时，会自动创建。当使用本地目录为源目录时，推荐使用COPY

#### ENTRYPOINT

格式：

```dockerfile
ENTRYPOINT ["executable","param1","param2"]
ENTRYPOINT command param1 param2 (shell中执行)
```

与CMD类似，配置容器启动后执行的命令，但不会被docker run 提供的参数覆盖。每个dockerfile只能有一个ENTRYPOINT，如果指定了多个，只有最后一个被执行

### 使用Dockerfile生成镜像

```bash
#mysoft/centos:6.6为新生成的镜像的标签，/tmp/dockerfile/centos6为dockerfile所在路径。
docker build -t mysoft/centos:6.6 /tmp/dockerfile/centos6   
```
