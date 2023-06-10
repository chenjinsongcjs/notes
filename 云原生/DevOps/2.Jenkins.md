# Jenkins

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_V83XT36SBs.png)

<https://www.jenkins.io/zh/>

## Jenkins的安装

```bash
/var/jenkins_home     是jenkins的家目录，包含了jenkins的所有配置和数据文件
以后要注意备份 /var/jenkins_home （以文件的方式固化的）

# Docker 方式安装命令
docker run \
-u root \
-d \
-p 8080:8080 \
-p 50000:50000 \
-v jenkins-data:/var/jenkins_home \
-v /etc/localtime:/etc/localtime:ro \
-v /var/run/docker.sock:/var/run/docker.sock \
--restart=always \
jenkinsci/blueocean

#自己构建镜像 RUN的时候就把时区设置好
#如果是别人的镜像，docker hub，UTC； 容器运行时 ， -v
/etc/localtime:/etc/localtime:ro


#jenkinsci/jenkins 是没有 blueocean插件的，得自己装
#jenkinsci/blueocean：带了的
#/var/run/docker.sock 表示Docker守护程序通过其监听的基于Unix的套接字。 该映射允许
#jenkinsci/blueocean 容器与Docker守护进程通信， 如果 jenkinsci/blueocean 容器需要实例化
#其他Docker容器，则该守护进程是必需的。 如果运行声明式管道，其语法包含agent部分用 docker；例
#如， agent { docker { ... } } 此选项是必需的。
#如果你的jenkins 安装插件装不上。使用这个镜像【 registry.cn-qingdao.aliyuncs.com/lfy/jenkins:plugins-blueocean 】默认访问账号/密码是
#【admin/admin】

#备份jenkins
tar -cvf jenkins_data.tar /var/lib/docker/volumes/jenkins-data/_data/
#恢复jenkins
tar -xvf jenkins_data.tar /var/lib/docker/volumes/jenkins-data/_data/

```

### 登录Jenkins

-   docker logs  Jenkins的名字    → 获取密码

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_0H6kgMmISR.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_feqeguoipj.png)

## Jenkins实战

代码在本地修改----提交到远程gitee----触发jenkins整个自动化构建流程（打包，测试，发布，部署）

-   Jenkins使用文档

<https://www.w3cschool.cn/jenkins/jenkins-qc8a28op.html>

### 准备一个git项目进行测试

1、idea创建Spring Boot项目
2、VCS - 创建git 仓库
3、gitee创建一个空仓库，示例为public
4、idea提交内容到gitee
5、开发项目基本功能，并在项目中创建一个Jenkinsfile文件
6、创建一个名为 devops-java-demo的流水线项目，使用项目自己的流水线

### Jenkins的工作流程

1、先定义一个流水线项目，指定项目的git位置

-   流水线启动
    1、先去git位置自动拉取代码
    2、解析拉取代码里面的Jenkinsfile文件
    3、按照Jenkinsfile指定的流水线开始加工项目

```palin
0、jenkins的家目录 /var/jenkins_home 已经被我们docker外部挂载了 ；
/var/lib/docker/volumes/jenkins-data/_data
1、WORKSPACE（工作空间）=/var/jenkins_home/workspace/java-devops-demo
  每一个流水线项目，占用一个文件夹位置
  BUILD_NUMBER=5；当前第几次构建
  WORKSPACE_TMP（临时目录）=/var/jenkins_home/workspace/java-devops-demo@tmp
2、常用的环境如果没有，jenkins配置环境一大堆操作
3、jenkins_url ： http://139.198.27.103:8080/

```

### 远程构建触发

期望效果： 远程的github代码提交了，jenkins流水线自动触发构建。

实现流程：
1、保证jenkins所在主机能被远程访问
2、jenkins中远程触发需要权限，我们应该使用用户进行授权
3、配置gitee/github，webhook进行触发

```plain
#远程构建即使配置了github 的webhook，默认会403.我们应该使用用户进行授权
1、在Jenkins中创建一个用户
2、一定随便登陆激活一次
3、生成一个apitoken
http://leifengyang:113620edce6200b9c78ecadb26e9cf122e@139.198.186.134:8080/job/dev
ops-java-demo/build?token=leifengyang
```

获取APIToken

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_fLzQQy2IQm.png)

远程构建

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_0LEWWsiwHU.png)

webhook

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_upugtlq82q.png)

## 流水线语法

<https://www.w3cschool.cn/jenkins/jenkins-qc8a28op.html>

```json
pipeline{
    agent any
    stages{
        stage("build"){
            steps{
                sh 'echo "build"'
                // 多条shell命令
                sh '''
                    echo "Multiline shell steps works too"
                    ls -alh
                '''
            }
        }
         stage("test"){
            steps{
                sh 'echo "test"'
            }
        }
         stage("run"){
            steps{
                sh 'echo "run"'
            }
        }
         stage("deploy"){
            steps{
                 //重试3次  直到成功或退出
                retry(3){
                    sh 'echo "test"'
                }
                //超时设置
                timeout(time: 3,unit: 'MINUTES'){
                    sh 'echo "timeout test"'
                    sh 'echo "retry"'
                }
                //“部署”阶段重试flakey-deploy.sh脚本3次，然后等待health-check.sh脚本执行最多3分钟。
                //如果健康检查脚本在3分钟内未完成，则Pipeline将在“部署”阶段被标记为失败。
                /*
                三分钟内重试5次
                timeout(time: 3, unit: 'MINUTES') {
                    retry(5) {
                        sh './flakey-deploy.sh'
                    }
                }
                */
            }
        }
    }
    //当pipeline执行完毕之后您可能需要运行清理步骤或根据Pipeline的结果执行某些操作。
    //可以在本post节中执行这些操作。
    post{
        always{
            echo "the will always run"
        }
        success{
            echo "the will run only if successful"
        }
        failure{
            echo "the will run only if failed"
        }
        unstable{
            echo 'This will run only if the run was marked as unstable'
        }
        changed{
             echo 'This will run only if the state of the Pipeline has changed'
             echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }

}
```

<https://www.jenkins.io/doc/book/pipeline/>
