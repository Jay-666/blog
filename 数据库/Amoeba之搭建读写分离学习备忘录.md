## 一.介绍
### Q1：什么是Amoeba？
Amoeba是mysql的代理应用，可以用Amoeba来灵活的代理mysql或mysql集群，客户端直接访问Amoeba服务就能访问到它所代理的mysql数据库了，访问方式是用mysql指令远程访问。  


### Q2:Amoeba能实现什么功能？
1.mysql集群的读写分离  
2.mysql集群的负载均衡  
3.mysql集群的数据切片  


### Q3: 搭建Amoeba重点关注是什么？
搞懂Amoeba的重点是搞懂三个配置文件：**dbserver.xml**、**amoeba.xml**、**rule.xml**。  
    1. dbserver.xml是用来告知Amoeba，需要被代理的mysql服务有哪些，配置这些mysql服务的ip地址、端口、登录帐号与密码等，然后还可定义多个mysql成一个集群，与集群轮转方式；  
    2. amoeba.xml是Amoeba的主配置文件，用来配置Amoeba的登录帐号、密码、端口号，和配置读服务器或读服务器集群、写服务器或写服务器集群；  
    3. rule.xml是用来配置数据切片。  

---


## 二.源码包搭建Amoeba读写分离
### **Amoeba代理Mysql读写分离，架构图**：  

客户  
｜  
｜  
 web应用程序，游戏程序（c,php,java.......)客户端  
 |  
 |  
代理层(Amoeba) 读写分离/数据切分  
 |  
 |  
mysql主 <----> mysql从*2  



### **安装前准备**：

1. amoeba-mysql-binary-2.2.0.tar.gz
2. jdk-8u161-linux-x64.tar.gz
3. 3台mysql服务器，并做一主二从
4. 1台用来搭建Amoeba的linux
5. 1台用来测试的Linux

**搭建过程**：

### **第一步：还是事前准备**

1.主机名三步，互相绑定
2.时间同步
3.关闭iptables,selinux

4. 配置好yum
5. 静态ip地址

### **第二步：安装Java**

因为Amoeba是用Java语言开发的，所以需要Java环境

1.安装jdk1.8版本，tar包源码包，解压即能用
```shell
[root@node1 ~]# ls
amoeba-mysql-binary-2.2.0.tar.gz jdk-8u161-linux-x64.tar.gz
[root@node1 ~]# tar xf jdk-8u161-linux-x64.tar.gz -C /usr/local
[root@node1 ~]# cd /usr/local/
[root@node1 local]# mv ./jdk1.8.0_161/ ./java   

```
验证一下刚解压安装的的jdk版本
```shell
[root@node1 local]# /usr/local/java/bin/java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```

### **第三步：安装amoeba软件**  

#### 1.将源码包解压至新建的目录

```shell
[root@node1 ~]# mkdir /usr/local/amoeba
[root@node1 ~]# tar xf /root/amoeba-mysql-binary-2.2.0.tar.gz -C /usr/local/amoeba
[root@node1 ~]# ls /usr/local/amoeba/
benchmark  bin  changelogs.txt  conf  lib  LICENSE.txt  README.html
```
关注下面二个目录  
bin 是启动脚本目录  
conf 是配置文件目录  
配置文件目录里我们要关注有三个文件:  
amoeba.xml--配置amoeba的全局配置文件  
dbServers.xml --配置amoeba连接mysql数据库的文件  
rule.xml--配置我们数据切分的文件  
    

2. #### 开始配置amoeba连接mysql数据库  
 
 
 
 ##### 2.1. 先修改dbserver.xml，配置mysql参数  
 
 
 ```shell
[root@node1 ~]# vim /usr/local/amoeba/conf/dbServers.xml 
 19                         <!-- mysql port 指定登录mysql的端口-->
 20                         <property name="port">3306</property>
 21 
 22                         <!-- mysql schema 指定访问的库,这里先填测试库-->
 23                         <property name="schema">test</property>
 24 
 25                         <!-- mysql user --指定登录用户名>
 26                         <property name="user">aa</property>
 27                         <!-- 指定登录密码-->
 28                         <property name="password">111</property>
 ...
 ...
 
  44         <dbServer name="server1"  parent="abstractServer">  --定义server1
 45                 <factoryConfig>
 46                         <!-- mysql ip -->
 47                         <property name="ipAddress">1.1.1.5</property> --mysql服务器的ip地址
 48                 </factoryConfig>
 49         </dbServer>
 50 
 51         <dbServer name="server2"  parent="abstractServer">  --定义server2
 52                 <factoryConfig>
 53                         <!-- mysql ip -->
 54                         <property name="ipAddress">1.1.1.6</property> --mysql服务器地址
 55                 </factoryConfig>
 56         </dbServer>
 57 
 58         <dbServer name="server3"  parent="abstractServer">  --定义server3
 59                 <factoryConfig>
 60                         <!-- mysql ip -->
 61                         <property name="ipAddress">1.1.1.7</property>  --mysql服务器ip地址
 62                 </factoryConfig>
 63         </dbServer>     
 64                         
 65         <dbServer name="multiPool" virtual="true">  --定义集群
 66                 <poolConfig class="com.meidusa.amoeba.server.MultipleServerPool">
 67                         <!-- Load balancing strategy: 1=ROUNDROBIN , 2=WEIGHTBASED , 3=HA-->
 68                         <property name="loadbalance">1</property>
 69                    
 70                         <!--server2、server3定义为集群multpool，它们为从服务器，等会儿把他们作为读服务-->
 71                         <property name="poolNames">server2,server3</property> 
 72                 </poolConfig>
 73         </dbServer>


 ```

保存退出

##### 2.2. 配置amoeba.xml

```shell
[root@node1 ~]# vim /usr/local/amoeba/conf/amoeba.xml 
 11   <property name="port">8066</property> --指定登录amoeba的端口号
 30   <property name="user">amoeba</property> --指定登录amoeba用户名
 32   <property name="password">123</property> --指定登录amoeba密码
<!--去掉注释-->
118                 <property name="writePool">server1</property> --指定server1为写服务器
119                 <property name="readPool">multiPool</property> --指定multPool为读集群

```

保存退出

3. #### 修改amoeba启动文件

启动文件/usr/local/amoeba/bin/amoeba 是需要jdk的支持才能启动，下面我们配置启动文件指定访问jdk
```shell
[root@node1 ~]# vim /usr/local/amoeba/bin/amoeba
 13 JAVA_HOME=/usr/local/java
 14 PATH=$PATH:$JAVA_HOME/bin
 15 export JAVA_HOME PATH
 ...
 ...
 62 DEFAULT_OPTS="-server -Xms256m -Xmx256m -Xss228k" --在62行把 Xss128k 改成228k，因启动时要求最低内存是228k 
```
注：  
-Xms256m --分配256m物理内存给amoeba软件用，连接数据库时初始化内存就要256m  
-Xmx256m --这个是amoeba软件最大可用的物理内存，（32位的JDK最大只能是2G，64位的JDK无限制但不能大于本机的物理内存大小）  
-Xss128k --默认是128k，但amoeba软件要求是228k，这个启动amoeba软件就要228k的内存

4. #### 再使用nohup方法启动Amoeba服务

```shell
nohup /usr/lcoal/amoeba/bin/amoeba start &
```
--这个启动方法把启动的信息写进nohup.out文件里，并在后台运行。建议用这种方法，方便我们排错。
停止服务的方法：nohup /usr/local/amoeba/bin/amoeba start &
___

### **第四步：在客户端测试**  

客户端需要安装有mysql，使用mysql命令连接Amoeba服务器登陆测试

```shell
mysql -uamoeba -p123321 -h1.1.1.7
mysql>
```

1. 如果有主从复制，则关闭；
2. 在Amoeba服务器的测试库插入一条数据，查看server1会有这条数据，server2、server3没有这条数据，用amoeba查询也没有这条数据；
3. 如果测试结果是这样的话，那就说明安装成功了；
4. 然后停止amoeba服务，进入amoeba.xml把测试库改成真正想要读写分离的库，删除测试库，开启主从复制；
5. 开启amoeba服务。