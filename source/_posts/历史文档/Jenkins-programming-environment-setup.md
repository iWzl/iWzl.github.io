---
title: Jenkins在CentOS系统环境上的搭建和部署
date: 2020-07-23 21:32:40
tag: 
    - DevOps
    - Liunx
    - Jenkins
    - CentOS
categories:
    - DevOps
---

### Jenkins服务在CentOS系统环境上的搭建和部署

> [Jenkins](https://www.jenkins.io/zh/) 由Java编写的一个开源的、提供友好操作界面的持续集成(CI)工具，起源于Hudson（Hudson是商用的），主要用于持续、自动的构建/测试软件项目、监控外部任务的运行（这个比较抽象，暂且写上，不做解释）。Jenkins用Java语言编写，可在Tomcat等流行的servlet容器中运行，也可独立运行。通常与版本管理工具(SCM)、构建工具结合使用。常用的版本控制工具有SVN、GIT，构建工具有Maven、Ant、Gradle。

<!-- more -->  

CI(Continuous integration，中文意思是持续集成)是一种软件开发时间。持续集成强调开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。借用网络图片对CI加以理解。

![CI](http://img.upuphub.com/6464255-1b6e3bfdbece1492.jpg)

 CD(Continuous Delivery， 中文意思持续交付)是在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境(类生产环境)中。比如，我们完成单元测试后，可以把代码部署到连接数据库的Staging环境中更多的测试。如果代码没有问题，可以继续手动部署到生产环境。下图反应的是CI/CD 的大概工作模式。

![Jenkins](http://img.upuphub.com/6464255-ba088ec7257062c0.jpg)

<!-- more -->

## 在线安装Jenkins

```shell
# 获取 repo
$ wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

# 获取key, 如果之前导入 jenkins 的key, 这一步可以忽略
$ rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

$ yum update && yum install jenkins

# 启用 jenkins
$ systemctl start jenkins
```

## 离线安装Jenkins

```shell
# 准备安装临时文件夹
$ mkdir jenkins
$ cd jenkins

# 获取最新的稳定版本的Jenkins.rpm文件(https://pkg.jenkins.io/redhat-stable/)
$ wget https://pkg.jenkins.io/redhat-stable/jenkins-2.235.2-1.1.noarch.rpm

# 执行本地的Yum安装
$ yum localinstall jenkins-2.235.2-1.1.noarch.rpm

# 启用 jenkins
$ systemctl start jenkins
```

## 安装异常解决

### 异常信息如下

```shell
[root@Leonardo-iWzl-Server jenkins]# systemctl start jenkins
Job for jenkins.service failed because the control process exited with error code. See "systemctl status jenkins.service" and "journalctl -xe" for details.
[root@Leonardo-iWzl-Server jenkins]# systemctl status jenkins
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: failed (Result: exit-code) since 四 2020-07-23 10:02:43 EDT; 16s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 23582 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=1/FAILURE)

7月 23 10:02:43 Leonardo-iWzl-Server systemd[1]: Starting LSB: Jenkins Automation Server...
7月 23 10:02:43 Leonardo-iWzl-Server runuser[23587]: pam_unix(runuser:session): session opened for user jenkins...d=0)
7月 23 10:02:43 Leonardo-iWzl-Server jenkins[23582]: Starting Jenkins bash: /usr/bin/java: No such file or directory
7月 23 10:02:43 Leonardo-iWzl-Server runuser[23587]: pam_unix(runuser:session): session closed for user jenkins
7月 23 10:02:43 Leonardo-iWzl-Server jenkins[23582]: [FAILED]
7月 23 10:02:43 Leonardo-iWzl-Server systemd[1]: jenkins.service: control process exited, code=exited status=1
7月 23 10:02:43 Leonardo-iWzl-Server systemd[1]: Failed to start LSB: Jenkins Automation Server.
7月 23 10:02:43 Leonardo-iWzl-Server systemd[1]: Unit jenkins.service entered failed state.
7月 23 10:02:43 Leonardo-iWzl-Server systemd[1]: jenkins.service failed.
Hint: Some lines were ellipsized, use -l to show in full.
```

### 原因说明

Jenkins 默认会在以下目录按顺序搜寻 JDK，一旦找到一个可用的即返回

```shell
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-1.7.0/bin/java
/usr/lib/jvm/jre-1.7.0/bin/java
/usr/lib/jvm/java-11.0/bin/java
/usr/lib/jvm/jre-11.0/bin/java
/usr/lib/jvm/java-11-openjdk-amd64
/usr/bin/java
```

如果系统的以上位置都未安装 JDK，启动时就会报错，可以通过建立软连接进行解决

```shell
 ln -s /usr/local/java/jdk1.8.0_261/bin/java /usr/bin/java
```

完成以上错误处理后,可得到以下输出

```shell
[root@Leonardo-iWzl-Server jenkins]# systemctl status jenkins
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: active (running) since 四 2020-07-23 10:10:06 EDT; 4s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 23606 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/jenkins.service
           └─23630 /usr/bin/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --dae...

7月 23 10:10:04 Leonardo-iWzl-Server systemd[1]: Starting LSB: Jenkins Automation Server...
7月 23 10:10:04 Leonardo-iWzl-Server runuser[23611]: pam_unix(runuser:session): session opened for user jenkins by (uid=0)
7月 23 10:10:06 Leonardo-iWzl-Server runuser[23611]: pam_unix(runuser:session): session closed for user jenkins
7月 23 10:10:06 Leonardo-iWzl-Server jenkins[23606]: Starting Jenkins [  OK  ]
7月 23 10:10:06 Leonardo-iWzl-Server systemd[1]: Started LSB: Jenkins Automation Server.
```

 查看jenkins 的启动参数 *ps -ef |grep jenkins* 

```shell
[root@Leonardo-iWzl-Server jenkins]# ps -ef |grep jenkins
jenkins  23630     1 49 10:10 ?        00:00:54 /usr/bin/java 
		-Dcom.sun.akuma.Daemon=daemonized 
		-Djava.awt.headless=true 
		-DJENKINS_HOME=/var/lib/jenkins 
		-jar /usr/lib/jenkins/jenkins.war 
		--logfile=/var/log/jenkins/jenkins.log 
		--webroot=/var/cache/jenkins/war 
		--daemon 
		--httpPort=8080 
		--debug=5 
		--handlerCountMax=100 
		--handlerCountMaxIdle=20
root     23695 23312  0 10:11 pts/0    00:00:00 grep --color=auto jenkins
```

在这里看到日志的一些配置和相关的端口、可以通过访问*「IP」:8080*访问到Jenkins服务

### Jenkins的更新

对于Jenkins更新可以通过

```shell
yum update jenkins
```

完成更新后,需要重启Jenkins服务

### Jenkins国内镜像配置

修改*/var/lib/jenkins* 目录下的*hudson.model.UpdateCenter.xml*文件内容如下

```xml
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <!-- <url>https://updates.jenkins.io/update-center.json</url>-->
     <url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json/url>
  </site>
</sites>
```

修改默认配置json文件*/var/lib/jenkins/updates/default.json*

```shell
# 进入响应文件夹
cd /var/lib/jenkins/updates

# 备份原始文件
cp default.json default.json.bak

# 替换更新下载地址
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json

# 替换测试URL
sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
```

完成以后重启Jenkins服务

### 配置Jenkins服务

之后按提示和Jenkins可视化引导完成剩余的Jenkins环境的搭建部署

## Jenkins的结构导图

![Jenkins](http://img.upuphub.com/6464255-cc56d3af1fdd96df.png)

---

## 参考和引用

[哥本哈根月光-Jenkins详细教程](https://www.jianshu.com/p/5f671aca2b5a)

[CentOS 安装 Jenkins 及 国内下载加速](https://halo.sherlocky.com/archives/jenkins)