# Docker命令

-   Docker的常见命令详解

<https://docs.docker.com/engine/reference/commandline/docker/>

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_fVIbOF-vk4.png)

| 命令        | 描述                                                                                 |
| --------- | ---------------------------------------------------------------------------------- |
| attach    | 绑定到运行中容器的 标准输入, 输出,以及错误流（这样似乎也能进入容器内容，但是一定小心，他们操作的就是控制台，控制台的退出命令会生效，比如 redis,nginx |
| build     | 从Dockerfile中构建镜像                                                                   |
| commit    | 把容器的改变提交为一个新的镜像                                                                    |
| cp        | 容器和宿主机之间的的文件和文件夹进行相互拷贝                                                             |
| create    | 创建一个容器，但是容器不启动，需要手动start或者stop。run命令是直接创建并运行                                       |
| diff      | 检查容器中的文件系统与原镜像的差别【A：添加文件或目录 D：文件或者目录删除 C：文件或者目录更改】                                 |
| events    | 获取服务器的实时事件                                                                         |
| exec      | 进入一个运行时容器内部                                                                        |
| export    | 导出容器的文件系统为tar文件commit是直接提交成镜像，export是导出成文件方便传输                                     |
| history   | 显示镜像的历史                                                                            |
| images    | 列出所有镜像                                                                             |
| import    | 导入tar文件的内容创建镜像，不能直接启动需要之前定义的完整命令才能启动。docker ps —no-trunc                           |
| info      | 显示系统信息                                                                             |
| inspect   | 获取docker组件的底层信息                                                                    |
| kill      | 杀死一个或者多个容器                                                                         |
| load      | 从tar文件加载镜像                                                                         |
| login     | 登录docker仓库                                                                         |
| logout    | 退出docker仓库                                                                         |
| logs      | 获取容器日志                                                                             |
| pause     | 暂停一个或者多个容器                                                                         |
| port      | 列出容器的端口映射                                                                          |
| ps        | 列出所有容器                                                                             |
| pull      | 从仓库中拉取镜像                                                                           |
| push      | 向仓库中推送镜像                                                                           |
| rename    | 为容器重命名                                                                             |
| restart   | 重启一个容器                                                                             |
| rm        | 删除一个或者多个容器                                                                         |
| rmi       | 删除一个或多个镜像                                                                          |
| run       | 创建并且启动一个容器                                                                         |
| save      | 把一个或者多个镜像保存为tar文件                                                                  |
| search    | 从docker hub中查找镜像                                                                   |
| start     | 启动一个或者多个镜像                                                                         |
| stats     | 显示容器资源的实时使用状态                                                                      |
| stop      | 停止一个或者多个容器                                                                         |
| tag       | 给原镜像创建一个标签，变为一个新的镜像                                                                |
| top       | 显示正在运行的容器的进程                                                                       |
| unpause   | pause的反向操作                                                                         |
| update    | 更新一个或者多个docker的配置                                                                  |
| version   | 获取docker版本信息                                                                       |
| container | 管理容器                                                                               |
| image     | 管理镜像                                                                               |
| network   | 管理网络                                                                               |
| volume    | 管理卷                                                                                |

```bash
# 1.attach 绑定监控标准的输入输出和错误流，是与控制台绑定的，执行推出操作是真的会退出
docker attach busybox

# 2.commit 将当前容器提交为新的镜像
docker commit nginx nginx:v1.
# 0REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
# nginx                 v1.0      0ad7281755f3   11 seconds ago   141MB
# nginx                 latest    f652ca386ed1   2 days ago       141MB

# 3.cp 将文件或者文件夹在容器和宿主机之间拷贝
docker cp nginx:/usr/share/nginx/html ./html  #将容器中的文件拷贝出来
docker cp ./index.html nginx:/usr/share/nginx/html # 将宿主机的文件拷贝到容器中

#4.create  创建一个容器但是不运行,和 run的区别就是不会运行，可以做一些cpu或者内存的限制
docker create --name nginx_c_test -p 80:80 nginx:latest docker create --name nginx_c_test -p 80:80 nginx:latest

# CONTAINER ID   IMAGE                 COMMAND                  CREATED              STATUS                     PORTS     NAMES
# 649ea7bd72f3   nginx:latest          "/docker-entrypoint.…"   About a minute ago   Created                              nginx_c_test

#5.diff 比较容器运行之后与镜像的差别
docker diff nginx
# 会描述每一层目录的变化
C /etc
C /etc/nginx
C /etc/nginx/conf.d
C /etc/nginx/conf.d/default.conf
C /usr
C /usr/local
A /usr/local/aegis
A /usr/local/aegis/aegis_client
A /usr/local/aegis/aegis_client/aegis_11_17
A /usr/local/aegis/aegis_client/aegis_11_17/data
A /usr/local/aegis/aegis_client/aegis_11_17/data/data.6

#6. events 监听docker服务器的事件  阻塞监听
docker events
# 2021-12-04T23:44:06.043734596+08:00 network connect 3ecda3dc4da7ef2780ff3c980ef5299fe47ff19ce0c97ff8e4f3001604e93d9d (container=be6f60260c44e0b381b51a36ab3ff2f95a2c6141fb1b0f2f3e8bbf9ed1ff904d, name=bridge, type=bridge)
# 2021-12-04T23:44:06.307279078+08:00 container start be6f60260c44e0b381b51a36ab3ff2f95a2c6141fb1b0f2f3e8bbf9ed1ff904d (image=nginx:latest, maintainer=NGINX Docker Maintainers <docker-maint@nginx.com>, name=nginx)

#7.exec 进入容器内部
docker exec -it nginx /bin/bash  # it 表示交互式进入，/bin/bash 表示进入的控制台 如：/bin/sh 等

# 8.export 将当前容器导出为他人文件， -o 指定导出的名称
docker export -o nginx.tar nginx

#9.history 显示镜像的历史，Dockerfile的建造过程
docker history nginx:latest 

IMAGE          CREATED      CREATED BY                                      SIZE      COMMENT
f652ca386ed1   2 days ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B        
<missing>      2 days ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B        
<missing>      2 days ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
<missing>      2 days ago   /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B        
<missing>      2 days ago   /bin/sh -c #(nop) COPY file:09a214a3e07c919a…   4.61kB    
<missing>      2 days ago   /bin/sh -c #(nop) COPY file:0fd5fca330dcd6a7…   1.04kB    
<missing>      2 days ago   /bin/sh -c #(nop) COPY file:0b866ff3fc1ef5b0…   1.96kB    
<missing>      2 days ago   /bin/sh -c #(nop) COPY file:65504f71f5855ca0…   1.2kB     
<missing>      2 days ago   /bin/sh -c set -x     && addgroup --system -…   61.1MB    
<missing>      2 days ago   /bin/sh -c #(nop)  ENV PKG_RELEASE=1~bullseye   0B        
<missing>      2 days ago   /bin/sh -c #(nop)  ENV NJS_VERSION=0.7.0        0B        
<missing>      2 days ago   /bin/sh -c #(nop)  ENV NGINX_VERSION=1.21.4     0B        
<missing>      2 days ago   /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B        
<missing>      2 days ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      2 days ago   /bin/sh -c #(nop) ADD file:ece5ff85ca549f0b1…   80.4MB   

#10.images 列出所有镜像
docker images  
两个相等
docker image ls

#11.import 将tar文件导入为镜像，，但是不能直接执行，需要原来的完整启动命令  docker ps --no-trunc
docker import nginx.tar nginx:v1.0
#sha256:c7dfc8c991f69b383cccea09aca5ed48103a934a57456d04a337283de626497e
#docker images
#REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
#nginx                 v1.0      c7dfc8c991f6   4 seconds ago   140MB
#nginx                 latest    f652ca386ed1   2 days ago      141MB

#12.info  显示系统信息
docker info
  
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Build with BuildKit (Docker Inc., v0.6.3-docker)
  scan: Docker Scan (Docker Inc., v0.9.0)
Server:
 Containers: 4
  Running: 1
  Paused: 0
  Stopped: 3
 Images: 5
 .......
 
#13.inspect 获取各种资源的底层信息
docker 容器/网络/镜像 。。。 inspect 资源名



```

```bash
#14.kill 杀死一个或多个容器
docker kill nginx  # 强制关闭，stop是优雅关闭，会等待任务处理完毕之后才关闭

#15.save 将镜像保存为压缩文件
docker save -o xxx.tar  镜像名

#16.load 将压缩文件加载为镜像
docker load -i xxx.tar

#17.login 、logout登录和登出docker仓库
docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: cjsstart
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded

docker logout
Removing login credentials for https://index.docker.io/v1/

#18.logs 查看日志
dokcer logs 容器名  # -f 选项追踪日志  -n 显示尾部几行

#19。pause  暂停一个或多个容器
docker pause 容器名
#20.unpause pause的反向操作
docker unpause 容器名
## 容器的几种状态 ，up、pauseed、create、exit

#21.port 列出容器的端口映射
docker port 容器名

docker port nginx 
80/tcp -> 0.0.0.0:80
80/tcp -> :::80

#22.ps  列出所有容器
docker ps  # -a 列出所有容器  -q列出容器id
[root@cjsstart ~]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS          PORTS                               NAMES
be6f60260c44   nginx:latest   "/docker-entrypoint.…"   15 hours ago   Up 16 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp   nginx

#23.pull  从仓库中拉取镜像
docker pull 镜像名:版本号  # 没有版本号默认为最新的

#24.push 向自己的仓库推送镜像，先登录自己的docker hub仓库，或者是其他的仓库
1.注册docker hub并登录
2.可以创建一个仓库，选为public 
3.docker push cjsstart/mynginx:tagname 
4.docker hub一个完整镜像的全路径是
5.官方完整路径docker.io/library/redis:alpine3.13 我们的 docker.io/cjsstart/mynginx:tagname 
6.docker images的时候镜像缩略了全名 默认官方镜像没有docker.io/library/ 
7.docker.io/ rediscommander / redis-commander:latest 
8.docker.io/cjsstart/mynginx:v4 我的镜像的全称
9.登录远程docker仓库
10.当前会话登录以后 docker login 。所有的东西都会push到这个人的仓库
11.docker push cjsstart/mynginx:tagname
12.上面命令的完整版 docker push docker.io/leifengyang/mynginx:v4
13.怎么知道是否登录了 cat ~/.docker/config.json 有没有 auth的值，没有就是没有登录
注：镜像需要改名为  名称空间/镜像名:版本号
docker tag 旧镜像  名称空间/镜像名:版本号
官方仓库太慢可以使用阿里云的

#25.rename 容器重命名
docker rename 容器名 新的名字

#26.restart 重启一个容器
docker restart 容器名

#27.rm 删除容器  rmi  删除镜像
docker rm -f 容器名
docker rmi -f 镜像名 
docker rm -f $(docker ps -aq) #删除所有容器
docker rmi -f $(docker images -aq) #删除所有镜像

#28 run 创建并运行一个容器
docker run -d --name xxx -p xx:xx 镜像 # 参数众多 ，--help查看

-d: 后台运行容器，并返回容器ID；
-i: 以交互模式运行容器，通常与 -t 同时使用； 
-P: 随机端口映射，容器内部端口随机映射到主机的端口 
-p:指定端口映射，格式为：主机(宿主)端口:容器端口 
-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用 
--name="nginx-lb":为容器指定一个名称； 
--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；
--dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；
-h "mars": 指定容器的hostname； 
-e username="ritchie": 设置环境变量； 
--env-file=[]: 从指定文件读入环境变量；
--cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行； 
-m :设置容器使用内存最大值； 
--net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型； 
--link=[]: 添加链接到另一个容器；
--expose=[]: 开放一个端口或一组端口；
--restart , 指定重启策略，可以写--restart=awlays 总是故障重启 
--volume , -v: 绑定一个卷。一般格式 主机文件或文件夹:虚拟机文件或文件夹

#29.search 从docker hub中查找镜像
[root@cjsstart ~]# docker search nginx
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                             Official build of Nginx.                        15917     [OK]       
jwilder/nginx-proxy               Automated Nginx reverse proxy for docker con…   2098                 [OK]
richarvey/nginx-php-fpm           Container running Nginx + PHP-FPM capable of…   820                  [OK]
jc21/nginx-proxy-manager          Docker container for managing Nginx proxy ho…   286                  
。。。。

#30 start 启动容器
docker start 容器名

#31 stats 查看当前容器的资源状态
docker stats 容器名

CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O           BLOCK I/O   PIDS
be6f60260c44   mynginx   0.00%     2MiB / 1.694GiB     0.12%     5.85kB / 4.06kB   0B / 0B     3

#32 stop 停止一个或多个容器
docker stop 容器名

#33 tag 给镜像打标签一般是推送镜像时使用
docker tag 镜像id  名称空间/镜像名:版本号

#34 top 显示正在运行的容器进程
docker top 容器名
 
#35 update  更新容器的配置  在这里可以对正在运行的容器做资源限制
docker update 容器名
#36 version 查看版本信息
docker version


```

-   镜像一般是：基础环境+软件组成，如：redis：linux+redis软件
-   制作镜像一般选择，基础环境较小的，如：带有slim、alpine标签的

## Docker 如何部署组件

-   从镜像仓库中查找镜像
-   查看文档
-   docker run创建并启动容器

### 常见的部署案例

-   部署nginx

```bash
# 注意 外部的/nginx/conf下面的内容必须存在，否则挂载会覆盖 
docker run --name nginx-app \
-v /app/nginx/html:/usr/share/nginx/html:ro \
-v /app/nginx/conf:/etc/nginx
-d nginx
```

-   部署MySQL

```bash
# 5.7版本 
docker run -p 3306:3306 --name mysql57-app \
-v /app/mysql/log:/var/log/mysql \
-v /app/mysql/data:/var/lib/mysql \
-v /app/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7 

#8.x版本,引入了 secure-file-priv 机制，磁盘挂载将没有权限读写data数据，所以需要将权限透传， 
#或者chmod -R 777 /app/mysql/data
# --privileged 特权容器，容器内使用真正的root用户 
docker run -p 3306:3306 --name mysql8-app \
-v /app/mysql/conf:/etc/mysql/conf.d \
-v /app/mysql/log:/var/log/mysql \
-v /app/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--privileged \
-d mysql

```

-   部署redis

```bash
# 提前准备好redis.conf文件，创建好相应的文件夹。如： port 6379 appendonly yes 
#更多配置参照 https://raw.githubusercontent.com/redis/redis/6.0/redis.conf 
docker run -p 6379:6379 --name redis \
-v /app/redis/redis.conf:/etc/redis/redis.conf \
-v /app/redis/data:/data \
-d redis:6.2.1-alpine3.13 \ 
redis-server /etc/redis/redis.conf --appendonly yes
```

-   部署ElasticSearch

```bash
#准备文件和文件夹，并chmod -R 777 xxx 
#配置文件内容，参照 https://www.elastic.co/guide/en/elasticsearch/reference/7.5/node.name.html 搜索相 关配置 
# 考虑为什么挂载使用esconfig ... 
docker run --name=elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms300m -Xmx300m" \
-v /app/es/data:/usr/share/elasticsearch/data \
-v /app/es/plugins:/usr/shrae/elasticsearch/plugins \
-v esconfig:/usr/share/elasticsearch/config \
-d elasticsearch:7.12.0
```

-   部署tomcat

```bash
# 考虑，如果我们每次 -v 都是指定磁盘路径，是不是很麻烦？ 使用挂载卷
docker run --name tomcat-app -p 8080:8080 \
-v tomcatconf:/usr/local/tomcat/conf \
-v tomcatwebapp:/usr/local/tomcat/webapps \
-d tomcat:jdk8-openjdk-slim-buster
```

### 重启策略

```bash
no，默认策略，在容器退出时不重启容器
on-failure，在容器非正常退出时（退出状态非0），才会重启容器
on-failure:3，在容器非正常退出时重启容器，最多重启3次 
always，在容器退出时总是重启容器
unless-stopped，在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器

```
