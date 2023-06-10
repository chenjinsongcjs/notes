# Docker存储原理

-   一个镜像可以启动多个容器，这些容器是依赖于这个镜像的基础环境的，所以不会占用太大的空间
-   Docker安装软件和宿主机安装软件的优缺点
    -   Docker安装移植性、便捷性高，可以做资源的隔离和限制
    -   Docker使用虚拟化技术，有一定的性能损失

## 镜像如何存储

-   镜像分层

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_N1w3Yln5za.png)

-   镜像存储(docker image inspect nginx) 分析nginx的存储管理

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_DwsZ_7JXL4.png)

-   LowerDir:底层目录

```bash
LowerDir:底层目录；diff只是存储的与原镜像的不同；包含基础的操作系统和软件
包含（可以进入每个目录查看）：用户目录、启动命令、配置文件、小型linux
Dockerfile的每个命令都会引起系统的改变 diff文件只是记录改变
我们exec 进入的容器就是容器对应镜像的小型操作系统
docker ps -s  # 可以看到每个容器的真实大小和虚拟大小
容器自己会建立层，如果想要修改东西，就会把想要修改的内容拷贝到容器层进行修改
```

-   MergedDir：合并目录；容器最终的完整工作目录全内容都在合并目录中；数据卷在容器层产生，所有的数据改变都在容器层

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_ajQOwKf35x.png)

-   UpperDir：上层目录：容器层
    -   容器删除之后容器层就消失，所有的改变都会消失
-   WorkDir：工作目录：临时层

> docker底层的 storage driver完成了以上的目录组织结果；

## Images and layers

Docker的镜像是由一系列的层构成每层代表Dockerfile的一条命令，除了最后一层外其他层都是只读的。

-   Dockerfile中有几条命令，镜像就有几层

描述

```docker
FROM ubuntu:15.04 
COPY . /app 
RUN make /app 
CMD python /app/app.py 
# 每一个指令都可能会引起镜像改变，这些改变类似git的方式逐层叠加。

#该Dockerfile包含四个命令，每个命令创建一个层。
#FROM语句从ubuntu：15.04映像创建一个图层开始。
#COPY命令从Docker客户端的当前目录添加一些文件。
#RUN命令使用make命令构建您的应用程序。
#最后，最后一层指定要在容器中运行的命令。
#每一层只是与上一层不同的一组。 这些层彼此堆叠。
#创建新容器时，可以在基础层之上添加一个新的可写层。 该层通常称为“容器层”。 对运行中
#的容器所做的所有更改（例如写入新文件，修改现有文件和删除文件）都将写入此薄可写容器层。

```

图示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_iiuAGfk_iZ.png)

## Container and Layers

描述

-   容器与镜像的区别在于可写顶层
-   在容器中添加新数据或者修改现有数据的所有写操作都存储在此可写层中
-   删除容器后，可写层也会被删除。基础镜像保持不变。因为所有的容器都有自己的可写容器层，并且将所有的改变都保存在该容器层中，所以多个容器可以共享同一个镜像，并且具有自己的数据状态

图示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_yAUKl54l1t.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_Q9o0ysMkgz.png)

## 磁盘容量预估

```bash
docker ps -s #可以显示出每个容器的占用磁盘大小
size: 表示每个容器可写层的数据量
virtual size：镜像大小+容器可写层的数据大小

多个容器可以共享部分或全部只读镜像数据。 
从同一镜像开始的两个容器共享100％的只读数据，
而具有不同镜像的两个容器（具有相同的层）共享这些公共层。 
因此，不能只对虚拟大小进行总计。这高估了总磁盘使用量，可能是一笔不小的数目。

```

## 镜像如何挑选

-   `busybox`：是一个集成了一百多个最常用`Linux`命令和工具的软件。`linux`工具里的瑞士军刀
-   `alpine：Alpine`操作系统是一个面向安全的轻型Linux发行版经典最小镜像，基于`busybox`，功能比`Busybox`完善。
-   `slim：docker` `hub`中有些镜像有`slim`标识，都是瘦身了的镜像。也要优先选择
-   无论是制作镜像还是下载镜像，优先选择`alpine`类型.

## Copy On Write （写时复制：读原文件，写复制文件）

描述

-   写时复制是一种共享和复制文件的策略，可最大程度地提高效率。
-   如果文件或目录位于镜像的较低层中，而另一层（包括可写层）需要对其进行读取访问，则它仅使用现有文件。
-   另一层第一次需要修改文件时（在构建镜像或运行容器时），将文件复制到该层并进行修改。 这样可以将`I / O`和每个后续层的大小最小化。

图

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_gsJzTjoeKJ.png)

## 容器如何挂载

描述

每个容器支持三种挂载：；

-   Volumes(卷) ：存储在主机文件系统的一部分中，该文件系统由Docker管理（在Linux上是“ / var /lib / docker / volumes /”）。 非Docker进程不应修改文件系统的这一部分。 卷是在Docker中持久存储数据的最佳方法。
-   Bind mounts(绑定挂载) 可以在任何地方 存储在主机系统上。 它们甚至可能是重要的系统文件或目录。 Docker主机或Docker容器上的非Docker进程可以随时对其进行修改。
-   tmpfs mounts(临时挂载) 仅存储在主机系统的内存中，并且永远不会写入主机系统的文件系统

图示

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_xi1XHzAIc2.png)

### volume（卷）

-   匿名卷的使用

```bash
docker run -dP -v :/etc/nginx nginx 
#docker将创建出匿名卷，并保存容器/etc/nginx下面的内容 
# -v 宿主机:容器里的目录
```

-   具名卷的使用

```bash
docker run -dP -v nginx:/etc/nginx nginx 
#docker将创建出名为nginx的卷，并保存容器/etc/nginx下面的内容
```

> 如果将一个空卷挂载到容器的目录或者文件中，那么容器中的内容会复制到这个空卷中&#x20;

> 如果启动容器并且指定一个卷挂载，但是这个卷不存在那么Docker会自动创建好这个卷

> \-v 宿主机绝对路径:Docker容器内部绝对路径：叫挂载；这个有空挂载问题&#x20;

> \-v 不以/开头的路径:Docker容器内部绝对路径：叫绑定（docker会自动管理，docker不会把他当前目录，而把它当前卷）

```bash
如果自己开发测试，用 -v 绝对路径的方式
如果是生产环境建议用卷
除非特殊 /bin/docker 需要挂载主机路径的则操作 绝对路径挂载

#卷：就是为了保存数据 
docker volume #可以对docker自己管理的卷目录进行操作； 
/var/lib/docker/volumes(卷的根目录)
docker volume inspect 卷名 # 查看卷的详细信息
```

## bind mount

```bash
如果将绑定安装或非空卷安装到存在某些文件或目录的容器中的目录中，则这些文件或目录会被
安装遮盖，就像您将文件保存到Linux主机上的/ mnt中一样，然后 将USB驱动器安装到/ mnt中。
在卸载USB驱动器之前，/ mnt的内容将被USB驱动器的内容遮盖。 被遮盖的文件不会被删除或更改，
但是在安装绑定安装或卷时将无法访问。
总结：外部目录覆盖内部容器目录内容，但不是修改。所以谨慎，外部空文件夹挂载方式也会导致容器内部是空文件夹

docker run -dP -v /my/nginx:/etc/nginx:ro nginx 
# bind mount和 volumes 的方式写法区别在于 
# 所有以/开始的都认为是 bind mount ，不以/开始的都认为是 volumes.

```

> 警惕bind mount 方式，文件挂载没有在外部准备好内容而导致的容器启动失败问题(空挂载)

## 管理卷

```bash
docker volume create xxx：创建卷名 
docker volume inspect xxx：查询卷详情 
docker volume ls: 列出所有卷 
docker volume prune: 移除无用卷
```

## docker cp的细节

> docker cp \[OPTIONS] CONTAINER:SRC\_PATH DEST\_PATH|- ：把容器里面的复制出来

> docker cp \[OPTIONS] SRC\_PATH|- CONTAINER:DEST\_PATH：把外部的复制进去

-   SRC\_PATH 指定为一个文件
    -   DEST\_PATH 不存在：文件名为 DEST\_PATH ，内容为SRC的内容
    -   DEST\_PATH 不存在并且以 / 结尾：报错
    -   DEST\_PATH 存在并且是文件：目标文件内容被替换为SRC\_PATH的文件内容。
    -   DEST\_PATH 存在并且是目录：文件复制到目录内，文件名为SRC\_PATH指定的名字
-   SRC\_PATH 指定为一个目录
    -   DEST\_PATH 不存在： DEST\_PATH 创建文件夹，复制源文件夹内的所有内容
    -   DEST\_PATH 存在是文件：报错
    -   DEST\_PATH 存在是目录
        -   SRC\_PATH 不以 /. 结束：源文件夹复制到目标里面
        -   SRC\_PATH 以 /. 结束：源文件夹里面的内容复制到目标里面

> 自己把文件目录创建好，避免未知错误
