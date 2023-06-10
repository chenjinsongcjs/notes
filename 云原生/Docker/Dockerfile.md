# Dockerfile

<https://docs.docker.com/engine/reference/builder/>

-   一般而言Dockerfile分为以下四个部分
    -   基础镜像信息
    -   维护者信息
    -   镜像操作信息
    -   启动时执行指令

## Dockerfile的基本命令

| 指令          | 说明                                                     |
| ----------- | ------------------------------------------------------ |
| FROM        | 指定基础镜像                                                 |
| MAINTAINER  | 指定维护者信息，已经过时，可以使用 LABLE maintainer=xxx来代替              |
| RUN         | 运行命令                                                   |
| CMD         | 指定容器启动时的默认命令                                           |
| ENTRYPOINT  | 指定容器的默认入口                                              |
| EXPOSE      | 指定镜像内服务监听的端口                                           |
| ENV         | 指定环境变量，可以在docker run 时通过-e指定值，会固化在image的config中        |
| ADD         | 复制指定src路径下的内容到容器中dest路径下，如果src是url则会自动下载，如果是tar文件会自动解压 |
| COPY        | 复制本地主机的内容到容器中指定的dest路径下，但是不会自动解压等                      |
| LABEL       | 指定生成镜像的元数据信息标签                                         |
| VOLUME      | 创建数据卷挂载点                                               |
| USER        | 指定运行容器是的用户名或UID                                        |
| WORKDIR     | 指定工作目录，为后续的RUN、CMD、ENTRYPOINT配置工作目录                    |
| ARG         | 指定镜像内使用的参数，可以在build的时候使用—buile-args改变值                 |
| OBBUILD     | 配置当创建的镜像为其他镜像的基础镜像时，所指定的创建操作指令                         |
| STOPSIGNAL  | 容器退出的信号值                                               |
| HEALTHCHECK | 健康检查                                                   |
| SHELL       | 指定使用shell时的默认类型                                        |

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_VlyEax6HmB.png)

## FROM

```[lain
FROM 指定基础镜像，最好挑一些apline，slim之类的基础小镜像. scratch镜像是一个空镜像，常用于多阶段构建
如何确定我需要什么要的基础镜像？
Java应用当然是java基础镜像（SpringBoot应用）或者Tomcat基础镜像（War应用）
JS模块化应用一般用nodejs基础镜像
其他各种语言用自己的服务器或者基础环境镜像，如python、golang、java、php等
```

## LABEL

```docker
标注镜像的一些说明信息。
LABEL multi.label1="value1" multi.label2="value2" other="value3" 
LABEL multi.label1="value1" \
 multi.label2="value2" \ 
 other="value3"

```

## RUN

-   RUN指令在当前镜像层顶部的新层执行任何命令，并提交结果，生成新的镜像层。
-   生成的提交映像将用于Dockerfile中的下一步。 分层运行RUN指令并生成提交符合Docker的核心概念，就像源代码控制一样。
-   exec形式可以避免破坏shell字符串，并使用不包含指定shell可执行文件的基本映像运行RUN命令。
-   可以使用SHELL命令更改shell形式的默认shell。 在shell形式中，您可以使用\（反斜杠）将一条RUN指令继续到下一行。

> RUN \<command> ( shell 形式, /bin/sh -c 的方式运行，避免破坏shell字符串)

> RUN \["executable", "param1", "param2"] ( exec 形式)

```docker
RUN /bin/bash -c 'source $HOME/.bashrc; \
 echo $HOME' 
#上面等于下面这种写法 
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME' 
RUN ["/bin/bash", "-c", "echo hello"] 123456
```

```docker
FROM alpine
LABEL maintainer="cjsstart"

ENV msg='hello world'

RUN echo "${msg}"
RUN ["echo","${msg}"] # 不是shell的形式不能获取变量
RUN ["/bin/bash","-c","echo ${msg}"]
CMD [ "sleep","10000" ]
#总结； 由于[]不是shell形式，所以不能输出变量信息，而是输出$msg。其他任何/bin/sh -c 的形式都 可以输出变量信息

shell和exec写法
1. shell 是 /bin/sh -c <command>的方式， 
2. exec ["/bin/sh","-c",command] 的方式== shell方式
 也就是exec 默认方式不会进行变量替换

```

## CMD和ENTRYPOINT

-   都可以作为容器启动入口

### 命令格式

```docker
# CMD的三种写法
CMD ["executable","param1","param2"] ( exec 方式, 首选方式) 
CMD ["param1","param2"] (为ENTRYPOINT提供默认参数) 
CMD command param1 param2 ( shell 形式)

# ENTRYPOINT的两种写法
ENTRYPOINT ["executable", "param1", "param2"] ( exec 方式, 首选方式) 
ENTRYPOINT command param1 param2 (shell 形式)

```

```docker
FROM alpine

LABEL maintainer="cjsstart"

CMD ["111111","222222"]  # 为ENTRYPOINT提供参数

ENTRYPOINT ["echo"]
```

### 一个Dockerfile中只能有一个CMD命令

-   Dockerfile中只能有一条CMD指令。 如果您列出多个CMD，则只有最后一个CMD才会生效。
-   CMD的主要目的是为执行中的容器提供默认值。 这些默认值可以包含可执行文件，也可以省略可执行文件，在这种情况下，您还必须指定ENTRYPOINT指令。

### CMD为ENTRYPOINT提供默认参数

-   如果使用CMD为ENTRYPOINT指令提供默认参数，则CMD和ENTRYPOINT指令均应使用JSON数组格式指定。

### 组合效果

|                                   | No ENTRYPOINT                | ENTRYPOINT exec\_entry p1\_entry | ENTRYPOINT \[“exec\_entry”, “p1\_entry”]           |
| --------------------------------- | ---------------------------- | -------------------------------- | -------------------------------------------------- |
| **No CMD**                        | *error, not allowed*         | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry                              |
| **CMD \[“exec\_cmd”, “p1\_cmd”]** | exec\_cmd p1\_cmd            | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry exec\_cmd p1\_cmd            |
| **CMD \[“p1\_cmd”, “p2\_cmd”]**   | p1\_cmd p2\_cmd              | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry p1\_cmd p2\_cmd              |
| **CMD exec\_cmd p1\_cmd**         | /bin/sh -c exec\_cmd p1\_cmd | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry /bin/sh -c exec\_cmd p1\_cmd |

### docker run启动参数会覆盖CMD内容

```dockr
FROM alpine

LABEL maintainer="cjsstart"

CMD ["111111","222222"]  # 为ENTRYPOINT提供参数

ENTRYPOINT ["echo"]

# 不给参数使用的是CMD的默认参数
docker run -it cmdtest:v1.0

#提供参数会覆盖CMD提供的参数
docker run -it cmdtest:v1.0 3333
```

## ARG和ENV

### ARG定义变量

-   ARG定义变量，用户可以在构建时使用—build-arg 指定变量的值，在docker build的时候传递给构建器
-   —build-arg指定参数会覆盖Dockerfile中的同名参数
-   如果用户指定了未在Docekrfile中定义的构建参数，则构建会输出警告
-   ARG只在构建期有效，运行期无效
-   不建议使用构建时变量来传递诸如github密钥、用户凭证等机密，因为构建时变量可以通过docker history查看
-   ARG定义的变量在Dockerfile中定义的行开始生效
-   使用ENV定义的环境变量始终会覆盖同名的ARG变量

### ENV

-   在构建阶段中所有后续指令的环境中使用，并且在许多情况下可以内联替换
-   引号和反斜杠可以用于在之中包含空格
-   ENV 可以使用key value的写法，但是这种不建议使用了，后续版本可能会删除

```docker
ENV MY_MSG hello 
ENV MY_NAME="John Doe" 
ENV MY_DOG=Rex\ The\ Dog
ENV MY_CAT=fluffy
#多行写法如下 
ENV MY_NAME="John Doe" MY_DOG=Rex\ The\ Dog \ 
MY_CAT=fluffy
```

-   docker run --env 可以修改这些值
-   容器运行时ENV值可以生效
-   ENV在image阶段会被解析并持久化，可以通过 docker image inspect 查看

## ADD和COPY

### COPY

```docker
COPY的两种写法
COPY [--chown=<user>:<group>] <src>... <dest> 
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]

```

-   \--chown功能仅在用于构建Linux容器的Dockerfiles上受支持，而在Windows容器上不起作用
-   COPY指令从src中复制新文件或目录，并将他们添加到容器的文件系统中，路径为dest
-   可以指定多个src源，但是文件和目录的路径将被解释为相对于构建上下文的源
-   每个 src 都可以包含通配符，并且匹配将使用Go的filepath.Match规则进行。

```docker
COPY hom* /mydir/ #当前上下文，以home开始的所有资源 
COPY hom?.txt /mydir/ # ?匹配单个字符 
COPY test.txt relativeDir/ # 目标路径如果设置为相对路径，则相对与 WORKDIR 开始 
# 把 “test.txt” 添加到 <WORKDIR>/relativeDir/ 

COPY test.txt /absoluteDir/  # 也可以使用绝对路径，复制到容器指定位置 

#所有复制的新文件都是uid(0)/gid(0)的用户，可以使用--chown改变 
COPY --chown=55:mygroup files* /somedir/ 
COPY --chown=bin files* /somedir/ 
COPY --chown=1 files* /somedir/ 
COPY --chown=10:11 files* /somedir/
```

### ADD

-   同COPY用法只是ADD拥有自动下载和解压的功能
-   src 路径必须在构建的上下文中； 不能使用 ../something /something 这种方式，因为docker构建的第一步是将上下文目录（和子目录）发送到docker守护程序。
-   如果 src 是URL，并且 dest 不以斜杠结尾，则从URL下载文件并将其复制到 dest 。如果 dest 以斜杠结尾，将自动推断出url的名字（保留最后一部分），保存到 dest
-   如果 src 是目录，则将复制目录的整个内容，包括文件系统元数据。

## WORKDIR和VOLUME

### WORKDIR

-   WORKDIR指令为Dockerfile中跟随它的所有 RUN，CMD，ENTRYPOINT，COPY，ADD 指令设置工作目录。 如果WORKDIR不存在，即使以后的Dockerfile指令中未使用它也将被创建。
-   WORKDIR指令可在Dockerfile中多次使用。 如果提供了相对路径，则它将相对于上一个WORKDIR指令的路径。

```docker
WORKDIR /a 
WORKDIR b 
WORKDIR c 
RUN pwd 
#结果 /a/b/c
```

-   也可以使用环境变量

```docker
ENV DIRPATH=/path 
WORKDIR $DIRPATH/$DIRNAME 
RUN pwd 
#结果 /path/$DIRNAME
```

### VOLUME

-   作用：把容器的某些文件夹映射到主机外部
-   写法

```docker
VOLUME ["/var/log/"] #可以是JSON数组 
VOLUME /var/log #可以直接写 
VOLUME /var/log /var/db #可以空格分割多个
```

> 用 VOLUME 声明了卷，那么以后对于卷内容的修改会被丢弃，所以， 一定在volume声明之前修改内容 ；

## USER

-   USER用来声明运行镜像时使用的用户名（UID）或可选组（GID）以及Dockerfile中USER后面所有RUN、CMD和ENTRYPOINT指令
-   写法

```docker
USER <user>[:<group>] 
USER <UID>[:<GID>]
```

## EXPOSE

-   EXPOSE指令通知Docker容器在运行时在指定的网络端口上进行侦听。 可以指定端口是侦听TCP还是UDP，如果未指定协议，则默认值为TCP。
-   EXPOSE指令实际上不会发布端口。 它充当构建映像的人员和运行容器的人员之间的一种文档，即有关打算发布哪些端口的信息。 要在运行容器时实际发布端口，请在docker run上使用-p标志发布并映射一个或多个端口，或使用-P标志发布所有公开的端口并将其映射到高阶端口。

```docker
EXPOSE <port> [<port>/<protocol>...] 
EXPOSE [80,443] 
EXPOSE 80/tcp 
EXPOSE 80/udp
```

## 多阶段构建

-   解决：如何让一个镜像变得更小;

<https://docs.docker.com/develop/develop-images/multistage-build/>

### 案例

```docker
#以下所有前提 保证Dockerfile和项目在同一个文件夹 
# 第一阶段：环境构建; 用这个也可以 
FROM maven:3.5.0-jdk-8-alpine AS builder 
WORKDIR /app 
ADD ./ /app 
RUN mvn clean package -Dmaven.test.skip=true 
# 第二阶段，最小运行时环境，只需要jre 第二阶段并不会有第一阶段哪些没用的层 
#基础镜像没有 jmap； jdk springboot-actutor（jdk） 
FROM openjdk:8-jre-alpine 
LABEL maintainer="534096094@qq.com" 
# 从上一个阶段复制内容 
COPY --from=builder /app/target/*.jar /app.jar 
# 修改时区 
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone && touch /app.jar 
ENV JAVA_OPTS="" 
ENV PARAMS="" 
# 运行jar包 
ENTRYPOINT [ "sh", "-c", "java -Djava.security.egd=file:/dev/./urandom $JAVA_OPTS -jar /app.jar $PARAMS" ]
```

```xml
<!--为了加速下载需要在pom文件中复制如下 -->
<repositories>
    <repository>
        <id>aliyun</id>
        <name>Nexus Snapshot Repository</name>
        <url>https://maven.aliyun.com/repository/public</url>
        <layout>default</layout>
        <releases>
            <enabled>true</enabled>
        </releases>        <!--snapshots默认是关闭的,需要开启 -->
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>aliyun</id>
        <name>Nexus Snapshot Repository</name>
        <url>https://maven.aliyun.com/repository/public</url>
        <layout>default</layout>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>
```

```docker
######小细节 
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone 或者
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone 可以让镜像时间同步。
 ## 容器同步系统时间 CST（China Shanghai Timezone）
-v /etc/localtime:/etc/localtime:ro 
#已经不同步的如何同步？ 
docker cp /etc/localtime 容器id:/etc/
```

## Images瘦身实践

-   选择最小的基础镜像
-   合并RUN环节的所有指令，少生成一些层
-   RUN期间可能安装其他程序会生成临时缓存，要自行删除。如：

```docker
# 开发期间，逐层验证正确的
 RUN xxx 
 RUN xxx 
 RUN aaa \ aaa \ vvv \ 
 #生产环境 
 RUN apt-get update && apt-get install -y \ 
 bzr \ 
 cvs \ 
 git \ 
 mercurial \ 
 subversion \ 
 && rm -rf /var/lib/apt/lists/*
```

-   使用 `.dockerignore `文件，排除上下文中无需参与构建的资源
-   使用多阶段构建
-   合理使用构建缓存加速构建。\[--no-cache]

## 更多Dockerfile写法

<https://github.com/docker-library/>
