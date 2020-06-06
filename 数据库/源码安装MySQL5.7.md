# 在Centos7源码包编译安装MySQL5.7

##   1、通过国内镜像下载源码包   

下载包含boost的源码包

```shell
wget http://mirrors.sohu.com/mysql/MySQL-5.7/mysql-boost-5.7.23.tar.gz
```

## 2、解压

```shell
tar xf mysql-boost-5.7.23.tar.gz
```

## 3、创建mysql用户

```shell
useradd -s/sbin/nologin mysql
```

## 4、创建数据库数据目录

```shell
mkdir -p /data/mysql/data
chown -R mysql:mysql /data/mysql
```

##   5、环境准备   

``` shell
yum install gcc gcc-c++ ncurses-devel perl autoconf cmake -y
```

##   6、编译-安装   

编译过程需要3~4g的内存,且过程比较漫长。

是虚拟机的话可以添加内存；

不选择加内存的话，可以新增临时的swap空间，用磁盘暂时代替内存   ，编译完后在删除临时的swap。

```shell
#开启临时swap分区
dd if=/dev/zero of=/swapfile bs=1M count=2048
mkswap /swapfile
swapon /swapfile
#进入源码包目录
cd mysql-5.7.23
#编译安装
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_BOOST=boost
make
make install

#关闭临时swap分区
swapoff /swapfile
rm /swapfile
```

##   7、编写配置my.cnf   

```shell
mkdir /usr/local/mysql/etc
vim /usr/local/mysql/etc/my.cnf

[mysqld]
user=mysql
port=3306
basedir = /usr/local/mysql
datadir=/data/mysql/data
socket=/tmp/mysql.sock
log-error=/data/mysql/mysql-error.log
pid-file=/data/mysql/mysql.pid
tmpdir=/tmp

[mysqld_safe]
log-error=/data/mysql/mysql-error.log
pid-file=/data/mysql/mysql.pid

[client]
socket=/tmp/mysql.sock
```

##   8、mysql初始化 

  ```shell
/usr/local/mysql/bin/mysqld   --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql/data --pid-file=/data/mysql/mysql.pid --tmpdir=/tmp
  ```

--initialize-insecure  root用户无密码

mysql安装好后可以用`mysqladmin -uroot passwd “新密码”`设置root密码

这一步容易报错，有报错可以看日志排错

##   9、拷贝mysql服务启动脚本并做修改

```shell
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
#修改/etc/init.d/mysqld中basedir、datadir、conf的值
sed -i "s|^basedir=.*|basedir=\/usr\/local\/mysql|" /etc/init.d/mysql
sed -i "s|^datadir=.*|datadir=\/data\/mysql\/data|" /etc/init.d/mysql
sed -i "s|conf=.*|conf=${INSTALL_DIR}\/mysql\/etc\/my.cnf|" /etc/init.d/mysql
```

##   10、把mysql命令添加环境变量中   

```shell
echo "export PATH=$PATH:/usr/local/mysql/bin" >>/etc/profile
source /etc/profile
```

## 11、启动mysql服务并登录

```shell
/etc/init.d/mysql start
#登录
mysql
```

