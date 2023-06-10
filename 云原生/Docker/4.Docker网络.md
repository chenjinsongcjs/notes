# Docker网络

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_UlHRbGCYQO.png)

## 端口映射

描述

-   一个宿主机的端口可以映射到容器的一个端口
-   容器的一个端口可以被宿主机的多个端口映射

```bash
docker create -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 --name hello-mysql mysql:5.7
```

-   \-p  宿主机端口:容器端口
-   \-P   随机映射到容器暴露端口

图
![image.png](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/20230611024306.png)

## 容器互联

> —link name:alias，name是连接的容器名称，alias是起的别名

-   场景：我们无需暴露MySQL的情况下，让web应用使用MySQL

```bash
docker run -d -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7 
docker run -d --link mysql01:mysql --name tomcat tomcat:7 
docker exec -it tomcat bash 

cat /etc/hosts 
# 172.17.0.3      mysql d61e52a5a00f mysql01

ping mysql
root@c2d7a0539ba3:/usr/local/tomcat# ping mysql01
PING mysql (172.17.0.3) 56(84) bytes of data.
64 bytes from mysql (172.17.0.3): icmp_seq=1 ttl=64 time=0.120 ms
64 bytes from mysql (172.17.0.3): icmp_seq=2 ttl=64 time=0.048 ms
64 bytes from mysql (172.17.0.3): icmp_seq=3 ttl=64 time=0.049 ms
64 bytes from mysql (172.17.0.3): icmp_seq=4 ttl=64 time=0.050 ms
64 bytes from mysql (172.17.0.3): icmp_seq=5 ttl=64 time=0.048 ms



```

> 容器互联缺点：将要访问的容器的ip写死到当前容器的/etc/hosts文件中，如果ip修改那么就不可访问。

## 自定义网络

### 默认网络原理

第1列

-   `Docker`使用`Linux`桥接，在宿主机虚拟一个`Docker`容器网桥(`docker0`)，`Docker`启动一个容器时会根据`Docker`网桥的网段分配给容器一个`IP`地址，称为`Container-IP`，同时`Docker`网桥是每个容器的默认网关。因为在同一宿主机内的容器都接入同一个网桥，这样容器之间就能够通过容器的`Container-IP`直接通信。
-   `Docker`容器网络就很好的利用了`Linux`虚拟网络技术，在本地主机和容器内分别创建一个虚拟接口，并让他们彼此联通（这样一对接口叫veth pair）；
-   `Docker`中的网络接口默认都是虚拟的接口。虚拟接口的优势就是转发效率极高（因为Linux是在内核中进行数据的复制来实现虚拟接口之间的数据转发，无需通过外部的网络设备交换），对于本地系统和容器系统来说，虚拟接口跟一个正常的以太网卡相比并没有区别，只是他的速度快很多。

第2列

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_vxv-hhOWrm.png)

原理：

1、每一个安装了Docker的linux主机都有一个docker0的虚拟网卡。桥接网卡

2、每启动一个容器linux主机多了一个虚拟网卡。

3、docker run -d -P --name tomcat --net bridge tomcat:8

### 网络模式

| 网络模式        | 配置                     | 说明                                                                      |
| ----------- | ---------------------- | ----------------------------------------------------------------------- |
| bridge模式    | —net=bridge            | 默认值，在Docker网桥docker0上为容器创建新的网络栈                                         |
| none模式      | —net=none              | 不配置网络，用户可以稍后进入容器自行配置                                                    |
| container模式 | —net=container:name/id | 容器和另外一个容器共享Network namespace。kubernetes中的pod就是多个容器共享一个Networknamespace。 |
| host模式      | —net=host              | 容器和宿主机共享Network namespace                                               |
| 用户自定义       | —net=mynet             | 用户自己使用network相关命令定义网络，创建容器的时候可以指定为自己定义的网络                               |

### 自建网络测试

```bash
#1、docker0网络的特点。， 默认、域名访问不通、--link 域名通了，但是删了又不行 
#2、可以让容器创建的时候使用自定义网络，用自定义 
  1、自定义创建的默认default "bridge" 
  2、自定义创建一个网络网络 
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet 
#所有东西实时维护好，直接域名ping通 
docker network connect [OPTIONS] NETWORK CONTAINER 
#3、跨网络连接别人就用。把tomcat加入到mynet网络 
docker network connect mynet tomcat 
  效果：1、自定义网络，默认都可以用主机名访问通 
       2、跨网络连接别人就用 docker network connect mynet tomcat 
       3.双方都加入自定义网络，就能通过容器访问
#4、命令 
1、容器启动，指定容器ip。 
docker run --ip 192.168.0.3 --net 自定义网络
2、创建子网。docker network create --subnet 指定子网范围 --driver bridge 所有东西实时 维护好，直接域名ping通 
3、docker compose 中的网络默认就是自定义网络方式。 docker run -d -P --network 自定义网络名(提前创建)

```

> 将容器加入同一个自定义网络就可以使用容器名进行访问
