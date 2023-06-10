# Docker基本概念

## Docker架构

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_yIeJIRAOlg.png)

-   Client：Docker的客户端，用于操作docker服务器（命令行或者界面）
-   DOCKER\_HOST：Docker主机，安装Docker服务的主机
-   Docker daemon：运行Docker后台服务的后台进程
-   Continers：Docker容器，一个容器对应一个实例，容器与容器之间资源隔离，还可以限制容器的资源
-   Images：Docker容器运行的基础镜像
-   Registry：Docker的镜像仓库，官方远程地址如下:

<https://hub.docker.com/>

> Docker用Go编程语言编写，并利用Linux内核的多种功能来交付其功能。 Docker使用一种称为名称空间的技术来提供容器的隔离工作区。 运行容器时，Docker会为该容器创建一组名称空间。 这些名称空间提供了一层隔离。 容器的每个方面都在单独的名称空间中运行，并且对其的访问仅限于该名称空间。

-   Docker中容器与镜像之间的关系：类比面向对象：镜像相当于模板类，容器相当于类的实例
-   容器与虚拟机的区别

区别

-   容器是Docker中的一个实例。他的资源被docker限制隔离，运行不需要分配大量的系统资源，而且内部集成了小型的Linux操作系统。
-   虚拟机是在宿主机上直接分配资源，资源没有得到很好的隔离，有可能出现一个虚拟机被另外的虚拟机占用资源的情况。而且每个虚拟机有着独立的完整的操作系统，耗费资源很大。

容器与虚拟机

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_Rau9nVn36v.png)

## Docker的隔离原理

-   namespace的6项隔离（资源隔离）

| namespace | 系统调用参数         | 隔离内容          |
| --------- | -------------- | ------------- |
| UTS       | CLONE\_NEWUTS  | 主机和域名         |
| IPC       | CLONE\_NEWIPC  | 信号量、消息队列和共享内存 |
| PID       | CLONE\_NEWPID  | 进程编号          |
| Network   | CLONE\_NEWNET  | 网络设备、网络栈、端口等  |
| Mount     | CLONE\_NEWNS   | 挂载点(文件系统)     |
| User      | CLONE\_NEWUSER | 用户和用户组        |

-   cgroups资源限制（资源限制）
    -   cgroup提供的主要功能如下：
        -   **资源限制**：限制任务使用的资源总额，并在超过这个 配额 时发出提示
        -   **优先级分配**：分配CPU时间片数量及磁盘IO带宽大小、控制任务运行的优先级
        -   **资源统计**：统计系统资源使用量，如CPU使用时长、内存用量等
        -   **任务控制**：对任务执行挂起、恢复等操作

> **cgroup**资源控制系统，每种子系统独立地控制一种资源。功能如下

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_30XstUAPxE.png)

## Docker安装

-   官方网站

<https://www.docker.com/>

-   Docker不同操作系统的安装方法

<https://docs.docker.com/get-docker/>

### 安装步骤：（CentOS）

#### 1、移除旧版本

```bash
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
或者
sudo yum remove docker*

```

#### 2、设置Docker的yum源

```bash
sudo yum install -y yum-utils
 sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
    国内可以使用阿里的源
sudo yum-config-manager \
  --add-repo \
   https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo


```

#### 3、安装最新的Docker（三选一）

```bash
 sudo yum install docker-ce docker-ce-cli containerd.io
```

#### 4、安装指定的版本（三选一）

```bash
# 1.找到所有可用的Docker版本
yum list docker-ce --showduplicates | sort -r
# 2.用上面查询到的版本号替换<VERSION_STRING> 填充版本号，
 sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io 
# 如： sudo yum install docker-ce-3:19.03.9-3.el7.x86_64 docker-ce-cli-3:19.03.9-3.el7.x86_64 containerd.io
# 注意" .x86_64 "这个大版本号
```

#### 5、离线安装（三选一）

```bash
 # 从这里下载安装包： https://download.docker.com/linux/centos/
 # sudo yum install /path/to/package.rpm
```

#### 6、启动服务

```bash
systemctl start docker #启动docker
systemctl enable docker #开机自启

# 测试安装结果
 sudo docker run hello-world

```

#### 7、配置镜像加速

```bash
# 从自己的阿里云获取镜像加速配置
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://4zanqsu0.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
 #以后docker下载直接从阿里云拉取相关镜像
```

> /etc/docker/daemon.json 是Docker的核心配置文件。

### 可视化界面-Portainer

<https://docs.portainer.io/v/ce-2.9/>

#### 安装

```bash
#创建挂载卷
docker volume create portainer_data
# 服务端部署 
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v
/var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce 
# 访问 9000 端口即可
#agent端部署 
docker run -d -p 9001:9001 --name portainer_agent --restart=always -v
/var/run/docker.sock:/var/run/docker.sock -v
/var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent
```

-   服务端下载说明

<https://docs.portainer.io/v/ce-2.9/start/install/server/docker/linux>

-   客户端下载说明

<https://docs.portainer.io/v/ce-2.9/start/install/agent/docker/linux>
