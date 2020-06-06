## jenkins的介绍和特点

1. 基于Java的的开源自动化平台
2. 提供IC/CD任务及流水线的服务
3. 丰富的插件系统，支持功能扩展
4. WEB的管理和使用界面
5. 开源免费
6. 支持分布式部署

## 1、确认Java环境

安装有jdk，正确的配置了JAVA_HMOE，且最好是1.8及以上，不然可能不兼容一些版本  

> 2.164（2019-02）及更高版本：Java 8或Java 11

> 2.54（2017-04）及更高版本：Java 8

> 1.612（2015-05）及更高版本：Java 7



## 2、下载安装

### 2.1、rpm安装
因为我网络环境不好，使用不了yum的方式，只能用rpm

```shell
wget https://pkg.jenkins.io/redhat-stable/jenkins-2.222.1-1.1.noarch.rpm
rpm -ivh jenkins-2.222.1-1.1.noarch.rpm
```



### 2.2、yum安装

```shell
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins
```





## 3、配置国内jenkins插件下载的镜像

在jenkins的家目录下找到**hudson.model.UpdateCenter.xml**

rpm安装的jenkins家目录是/var/lib/jenkins

```xml
<?xml version='1.1' encoding='UTF-8'?>
<sites>
<site>
<id>default</id>
<url>https://updates.jenkins.io/update-center.json</url>
</site>
</sites>
```


把https://updates.jenkins.io/update-center.json

改成http://mirror.xmission.com/jenkins/updates/update-center.json或清华的镜像http://updates.jenkins.io/update-center.json

## 4、启动服务

启动前确认没有其他的服务在占用8080端口

```shell
systemctl daemon-reload
systemctl start jenkins
```

`netstat -ntlup|grep` 8080 #8080端口起来了就说明启动成功



**报错1**：jenkins[19208]: Starting Jenkins bash: /usr/bin/java: 没有那个文件或目录

原因：因为是二进制安装的Java，所以没找到/usr/bin/java这个文件

解决方法1：修改配置jenkins文件

```sehll
vim /etc/init.d/jenkins
...

candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-1.7.0/bin/java
/usr/lib/jvm/jre-1.7.0/bin/java
/usr/lib/jvm/java-11.0/bin/java
/usr/lib/jvm/jre-11.0/bin/java
/usr/lib/jvm/java-11-openjdk-amd64
/usr/local/jdk1.8.0/bin/java
"
...
systemctl daemon-reload
systemctl start jenkins
```



解决方法2：创建软连接

`ln -s /usr/local/jdk1.8.0_161/bin/java /usr/bin/java`





## 5、访问jenkins页面

用浏览器访问jenkins服务器的  IP:8080

在下载默认插件时卡住；卡在打开网站后 Please wait while Jenkins is getting ready to work ...都是因为没有配置好国内镜像

1）解锁Jenkins

![img](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/%E8%A7%A3%E9%94%81jenkins.png)

在jenkins服务器上执行`cat /var/lib/jenkins/secrets/initialAdminPassword`就能得到密码，密码文件也会在你使用后删除

2）设置用户

3）插件下载

下载默认的插件就行了，之后可以需要什么插件再下载

4）界面介绍

![img](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/index.png)

| 单元               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| 新建任务           | 创建一个具体的任务，具体怎么去执行构建部署                   |
| 用户列表           | 用户管理，修改及创建用户或者密码                             |
| 构建历史           | 每个项目的构建历史记录                                       |
| 系统管理           | Jenkins的一些基本配置（比较重要的配置：系统设置、凭证配置、插件管理、节点管理、管理用户） |
| 我的视图           | 此用户拥有权限的视图（视图：多个项目的集合）                 |
| 凭证               | 较为机密的用户密码及认证信息存储地                           |
| Lockable Resources | jenkins的锁定资源                                            |
| 新建视图           | 新建一个视图，供你新建任务                                   |
| 构建队列           | 正在构建的任务列表                                           |
| 构建执行状态       | 构建任务状态或者节点状态                                     |

## **6、新建第一个项目hello world**

通过第一个没什么正式用图的hello world项目，了解一些jenkins项目是怎么创建的

1）点击新建任务，填写项目名，点确认

![img](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/hello-world-1.png)

2）填写描述；在项目里添加一个String类型的变量，等会在构建shell脚本中打印出来

![img](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/hello1.png)

3）在底部的构建栏中选择 shell脚本的方式构建

并写入两行shell语句，并点保存

echo "hello world"

echo "$path1"

如图

![img](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/hello2.png)

4）保存后自动跳转到hello world项目界面，点击“ Build with Parameters”，开始执行构建

![img](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/%E6%9E%84%E5%BB%BA.png)

5）点击最新的构建历史记录

![img](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/hello%231.png)

再点控制台输出，看到我们刚刚写的shell脚本的打印结果，这就说明hello world执行构建成功

![img](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/202005/jenkins/hello%E7%BB%93%E6%9E%9C.png)

---

思考：既然jenkins能执行shell，那我把拉取代码 ->构建 -> 发布 -> 测试网站->回滚备份......一系列操作写成shell脚本就行啦，为什么还需要jenkins？