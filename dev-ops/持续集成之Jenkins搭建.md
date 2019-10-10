#### 什么是持续集成？

​		一般部署项目大致分为这几个步骤:  编译、测试、打包、部署，这个是重复的过程，耗时耗力，持续集成就是利用工具把这个过程自动化。

- 持续集成（Continuous Integration）

  ![持续集成](https://pic1.zhimg.com/80/v2-74003ecc20c5e94a2d028c70b2ec35b7_hd.jpg)

- 持续交付（Continuous Delivery）

  ![持续交付](https://pic4.zhimg.com/80/v2-2dcc7b4b54f5951fd797c87f5a09f9d3_hd.jpg)

- 持续部署（Continuous Deployment）

  ![持续部署](https://pic4.zhimg.com/80/v2-3074a37c46a00c6a861519ba5725fa04_hd.jpg)

#### 相关概念

[如何理解持续集成、持续交付、持续部署？](https://www.zhihu.com/question/23444990)

[阮一峰-持续集成是什么？](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)

#### 常见持续集成工具

- [Jenkins](https://jenkins.io/zh/)
- [GitLab CI](https://about.gitlab.com/product/continuous-integration/)
- [Travis CI](https://www.travis-ci.org/ )
- [TeamCity](www.jetbrains.com/team... )

#### 搭建jenkins，这里介绍三种方式

[jenkins中文官网](https://jenkins.io/zh/)

一、使用war包(最简单的方式)

1. 下载稳定版本的war包，推荐[清华源](https://mirrors.tuna.tsinghua.edu.cn/jenkins/)

   ```shell
   wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/war-stable/2.176.3/jenkins.war
   ```

2. 启动jenkins

   ```shell
   # 需要安装java环境
   java -jar  jenkins.war --httpPort=5555
   
   #使用tmux后台启动服务
   
   # 安装
   sudo apt-get install tmux
   
   # 新建session
   tmux new -s jenkins
   
   # 断开会话，服务继续运行
   使用快捷键组合Ctrl+b+d，应该是ctrl+b松开后再按其他键。
   例如ctrl+b d，应该先同时按ctrl+b 松开后，再按 d
   
   #查看会话列表
   tmux ls
   
   # 进入会话
   tmux a -t jenkins
   ```

   [Tmux详细介绍](http://louiszhai.github.io/2017/09/30/tmux/#%E5%AE%89%E8%A3%85)

3. 登录、安装插件

   - 根据提示获取管理员密码并登录

   ![](https://dingxu66.oss-cn-beijing.aliyuncs.com/img/20190912110325.png)

   - 安装插件(可能会因为网络原因安装失败，直接重新安装)

     ![plugins](https://dingxu66.oss-cn-beijing.aliyuncs.com/img/3F6D45F1-EA78-45CB-A92B-8CDDAE698806.png)

   - 然后下一步提示创建一个管理员账号，填写完就进入jenkins首页。

     ![homepage](https://dingxu66.oss-cn-beijing.aliyuncs.com/img/76CBA9E6-BCE9-403D-8BBD-40958DF07932.png)

4. 创建项目、配置

   - 本地项目编写Jenkinsfile（这里介绍Pipeline方式构建）

     ```groovy
     pipeline {
     	agent any
     	options {
     		retry(3)
     		timeout(time: 10, unit: 'MINUTES')
     	}
     	stages {
     		stage('start') {
     			steps {
     				echo "Let's Go"
     			}
     		}
     		stage('build') {
     			when {
     				branch 'dev'
     			}
     			steps {
                     sh '开始构建....'
     				// sh 'sh deploy.sh'
     			}
     		}
     	}
     	post {
     		success {
     			echo 'jenkins构建成功'
     		}
     		failure {
     			echo 'jenkins构建失败'
     		}
     	}
     }
     ```

   - 然后在jenkins首页，打开blue ocean

     ![blue ocean](https://dingxu66.oss-cn-beijing.aliyuncs.com/img/82D89327-DC02-429D-90B4-ED72BA76EB79.png)

   - 选择git代码仓库，填入git地址

     ![git repository](https://dingxu66.oss-cn-beijing.aliyuncs.com/img/233C82E0-06C0-49A8-92ED-28537474B845.png)

   - 如果选择GitHub项目需要配置token（User-Settings-Developer settings-tokens）,然后勾选repo、user。

     ![github](https://dingxu66.oss-cn-beijing.aliyuncs.com/img/20190912114550.png)

   - 其他Git项目就填入地址后根据提示需要将jenkins服务器的ssh公钥key配置到git服务器上，然后创建流水线。创建后git项目根目录有Jenkinsfile的自动构建。

   - 在项目配置中，选择扫描项目的时间间隔，分支代码有改动自动触发构建

     ![scan](https://dingxu66.oss-cn-beijing.aliyuncs.com/img/119A93DA-20DD-404F-8AA1-5AB37CBACFCC.png)

二、apt-get源安装

[安装参考jenkins中文官网](https://jenkins.io/zh/doc/book/installing/)

1. 更新apt源并安装jenkins

   ```shell
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
   
   sudo apt-get update
   sudo apt-get install jenkins
   ```

2. 可以在`/etc/default/jenkins`查看或修改 jenkins的配置选项，比如启动用户、启动端口、数据目录等。

   ```shell
   NAME=jenkins
   
   # 数据目录
   JENKINS_HOME=/var/lib/$NAME
   
   # 日志文件
   JENKINS_LOG=/var/log/$NAME/$NAME.log
   
   # jenkins启动用户，默认为jenkins
   JENKINS_USER=$NAME
   
   # pid
   PIDFILE=/var/run/$NAME/$NAME.pid
   
   # war包位置
   JENKINS_WAR=/usr/share/$NAME/$NAME.war
   
   # 端口，默认8080
   HTTP_PORT=8080
   ```

3. 默认会将Jenkins设置为守护进程。并创建一个jenkins用户来运行此服务。

4. 停止、启动jenkins

   ```shell
   /etc/init.d/jenkins stop
   /etc/init.d/jenkins start
   
   service jenkins stop
   service jenkins start
   ```

5. 升级jenkins

   ```text
   apt-get安装方式简化了软件安装以及软件启动和停止，但实际就是用内置的web服务加上jenkins.war，默认情况下这个文件在/usr/share/jenkins/ 目录，如果要对其升级，只需要替换这个文件即可。
   
   为了保险起见，你可以备份这个文件，然后在 jenkins下载页面找到 Generic Java package (.war) ，把下载下来的文件 放到/usr/share/jenkins/ 目录，下载好了重新启动完成升级操作。
   ```

   

三、docker安装jenkins

​	[docker安装参考链接](https://yeasy.gitbooks.io/docker_practice/
)

1. 卸载旧版本docker

   ```shell
   sudo apt-get remove docker \
                  docker-engine \
                  docker.io
   ```

2. 安装docker

   ```shell
   curl -fsSL get.docker.com -o get-docker.sh
   
   sudo sh get-docker.sh --mirror Aliyun
   ```

3. 启动docker

   ```shell
   # 设置开启自启动
   sudo systemctl enable docker
   
   # 启动docker
   sudo systemctl start docker
   
   # 查看版本
   docker -v    # docker --version
   ```

4. 安装docker-compose

   ```shell
   sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   
   # 添加执行权限
   sudo chmod +x /usr/local/bin/docker-compose
   
   # 查看版本
   docker-compose -v    # docker-compose --version
   ```

5. 配置docker镜像加速

   ```shell
   在/etc/docker/daemon.json 中写入如下内容，如果没有此文件新建即可:
   {
     "registry-mirrors": [
       "https://dockerhub.azk8s.cn",
       "https://reg-mirror.qiniu.com"
     ]
   }
   
   重新启动服务:
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   
   检查加速器是否生效:
   docker info
   
   如果出现下面内容则配置成功
   Registry Mirrors:
    https://dockerhub.azk8s.cn/
    https://reg-mirror.qiniu.com/
   ```

6. 安装jenkins，使用blue ocean镜像

   ```shell
   docker run \
       --name jenkins \
       -u root \
       -d \
       -p 5090:8080 \
       -v /home/root/jenkins_home:/var/jenkins_home \
       -v /home/root/.ssh:/root/.ssh \
       --restart=always \
       jenkinsci/blueocean
       
   # 将jenkins的数据目录挂载出来，防止数据丢失
   # 这里把.ssh目录挂载出来的原因是:
   
   1.如果需要将应用部署到其他服务器上，需要通过ssh认证来远程部署应用，所以只需要保证主机host和部署服务器连通，而不用再到jenkins容器中生成ssh密钥，来保证容器与部署服务器连通性。
   
   2.将ssh公钥传入到git服务器上，保证能正常拉取代码
   
   ```

7. 查看容器日志

   ```shell
   ➜  ~ sudo docker ps
   CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                               NAMES
   a210c6d643a3        jenkinsci/blueocean   "/sbin/tini -- /usr/…"   3 minutes ago       Up 3 minutes        50000/tcp, 0.0.0.0:8090->8080/tcp   jenkins
   
   # 查看jenkins日志
   ➜  ~ sudo docker logs -f --tail=50 a210c6d643a3
   ```


## Author

👤 **JohnnyTing**

* Github: [@JohnnyTing](https://github.com/JohnnyTing)