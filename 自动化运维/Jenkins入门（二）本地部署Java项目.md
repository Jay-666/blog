目的：在同一台服务器上用Jenkins+git+maven自动化构建部署Java项目

### 1、环境准备

一台服务器上搭建有：

1. tomcat
2. Java
3. git
4. [maven](https://www.cnblogs.com/ningzijie/p/12832672.html)
5. [jenkins](https://www.cnblogs.com/ningzijie/p/12839723.html)

### 2、在jenkins上新建项目

![1](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo1.png)

填入项目名

![2](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo2.png)

随便填写写描述；

点击git，在 Repository URL中填入github项目，我这里填的是阿良老师的一个Java demo项目：https://github.com/lizhenliang/tomcat-java-demo.git

![3](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo3.png)

![4](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo4.png)



在底部的Excuse shell写入

```shell
#!/bin/bash
sh -x /opt/jenkins-tomcat.sh
```



![5](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo5.png)

### 3、编写shell脚本

在opt/中新建一个shell脚本，并修改对应自己服务器的变量值

`vim /opt/jenkins-tomcat.sh`

```shell
#!/bin/bash
#需要 root权限 git maven /data/backup

#备份文件后缀
DATE=$(date +%F_%T)
#tomcat目录名
TOMCAT_NAME=tomcat8.0
#tomcat完整路径
TOMCAT_DIR=/usr/local/$TOMCAT_NAME
#tomcat  ROOT路径
ROOT=${TOMCAT_DIR}/webapps/ROOT

#备份存放路径
BACKUP_DIR=/data/backup
#jenkins存放git拉取代码的路径
WORK_DIR=/var/lib/jenkins/workspace
#项目名
PROJECT_NAME=Java-test
#maven的家目录
MAVEN_HOME=/usr/local/maven3.6

#防止jenkins默认shell执行完后，终止其子进程
BUILD_ID=DONTKILLME

#构建
cd ${WORK_DIR}/${PROJECT_NAME}
${MAVEN_HOME}/bin/mvn clean
if [ $? -ne 0 ]; then
  echo "maven bauid failure!"
  exit 1
fi
${MAVEN_HOME}/bin/mvn package
if [ $? -ne 0 ]; then
  echo "maven bauid failure!"
  exit 1
fi


#部署
TOMCAT_PID=$(ps -ef|grep "$TOMCAT_NAME"|egrep -v "grep|$0" |awk 'NR==1{print $2}' )
#ps $TOMCAT_PID
[ $TOMCAT_PID!="" ] && kill -9 $TOMCAT_PID
#[ $TOMCAT_PID!="" ] && $TOMCAT_DIR/bin/shutdown.sh

[ -d $ROOT ]&& mv $ROOT $BACKUP_DIR/${TOMCAT_NAME}_ROOT$DATE
cp -f  $WORK_DIR/$PROJECT_NAME/target/*.war   ${ROOT}.war
$TOMCAT_DIR/bin/startup.sh


TOMCAT_NEW_PID=$(ps -ef|grep "$TOMCAT_NAME"|egrep -v "grep|$$" |awk 'NR==1{print $2}' )
echo "启动成功，tomcat pid:${TOMCAT_NEW_PID}"
~                                                   

```



建立备份目录

```shell
mkdir -p /data/backup
```

### 4、配置

 #### 4.1、配置jenkins的Java路径

![6](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo6.png)



填入你服务器上的Java安装路径，然后保存

![7](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo7.png)

#### 4.2、修改jenkins执行用户

为了防止执行shell的时候出现“权限不足”的情况，所以把jenkins的执行用户改成root

```shell
vim /etc/sysconfig/jenkins 

#把JENKINS_USER的值还成root
JENKINS_USER="root"
```





### 5、执行Java-demo项目

![8](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo8.png)

点击你的项目名

![9](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo9.png)

执行构建后，进入控制台

![10](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo10.png)

出现下面两句就是构建成功了，如果构建失败，也会把报错打印出来

![11](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo11.png)

访问tomcat网站页面

![12](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/Java-demo12.png)



（完）

### 5、BUG记录：

#### BUG1：:执行mvn报错没有找到jdk

解决：在系统设置—>全局工具配置—>设置Java的家目录



#### BUG2:执行shell的时候权限不足

解决1：修改jenkins服务的默认用户为root

解决2：根据具体情况赋予部分权限给jenkins



#### BUG3：在构建时，tomcat提起来了，构建完成后tomcat却又没提起来

分析：jenkins用shell构建完成后，其子进程也随之被销毁

解决1：后台启动tomcat

nohup sh /usr/local/tomcat8.0/bin/startup.sh &

解决2：在脚本前面加入

```shell
BUILD_ID=DONTKILLME
```

#### BUG4：构建成功却报错

```shell
Build step 'Execute shell' marked build as failure
Finished: FAILURE
```

解决：检查代码，看是否有语法错误