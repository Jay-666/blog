## 搭建两台NFS服务器

| 用途               | 主机         |
| ------------------ | ------------ |
| 主NFS服务器        | 192.168.0.15 |
| 从NFS服务器 - 备份 | 192.168.0.16 |
| 测试服务器         | 192.168.0.13 |



两台NFS做一样的操作，搭建NFS服务器

### 第一步：安装NFS软件

```shell
yum  install nfs* -y
```

### 第二步：创建nfsnobody用户

```shell
# 查看是否有nfsnobody用户
[root@localhost ~]#id nfsnobody
uid=65534(nfsnobody) gid=65534(nfsnobody) 组=65534(nfsnobody)

#没有的话，再自己创建
[root@localhost ~]#useradd -u 12306 -s /sbin/nologin -M nfsnobody
```

### 第三步：设置共享目录/data的属主和属组为指定用户

```shell
[root@localhost ~]# mkdir /data
[root@localhost ~]# chown -R nfsnobody.nfsnobody /data
```

### 第四步：修改/etc/exports配置文件

```shell
[root@localhost ~]#echo "/data 192.168.0.0/24(rw,sync,all_squash,anonuid=65534,anongid=65534)" >> /etc/exports 
```

### 第五步：启动NFS

```shell
[root@localhost ~]# systemctl start nfs
[root@localhost ~]# systemctl enable nfs
```

修改配置文件后需要重启nfs服务或使用命令：`# exportfs -arv `重新加载配置文件

### 第六步：挂载测试

在192.168.0.13上，挂载测试

```shell
[root@localhost mnt]# yum -y install nfs-utils

# 显示NFS服务器上所有的共享目录
[root@localhost mnt]# showmount -e 192.168.0.15
Export list for 192.168.0.15:
/data 192.168.0.0/24
[root@localhost mnt]# showmount -e 192.168.0.16
Export list for 192.168.0.16:
/data 192.168.0.0/24

# 创建目录，并挂载上nfs共享目录
[root@localhost mnt]# mkdir /mnt/nfs{1,2}
[root@localhost mnt]# mount -t nfs 192.168.0.15:/data /mnt/nfs1
[root@localhost mnt]# mount -t nfs 192.168.0.16:/data /mnt/nfs2

# 查看是否挂载上
[root@localhost mnt]# df -h
文件系统             容量  已用  可用 已用% 挂载点
/dev/mapper/cl-root   19G  5.2G   14G   28% /
devtmpfs             1.4G     0  1.4G    0% /dev
tmpfs                1.4G     0  1.4G    0% /dev/shm
tmpfs                1.4G  8.7M  1.4G    1% /run
tmpfs                1.4G     0  1.4G    0% /sys/fs/cgroup
/dev/sr0             7.8G  7.8G     0  100% /yum
/dev/sda1            197M  117M   81M   60% /boot
tmpfs                284M     0  284M    0% /run/user/0
192.168.0.16:/data    19G  1.9G   17G   10% /mnt/nfs2
192.168.0.15:/data    19G  7.0G   12G   37% /mnt/nfs1


```

取消挂载命令：`umount 192.168.0.15:/data`



## 搭建rsync+inotify实现NFS单向同步

### 第一步：在两台NFS上安装同版本rsync

```shell
# 两台服务器的yum源要一致
[root@localhost mnt]# yum install -y rsync
# 查看rsync版本是否一直
[root@localhost data]# rsync --version
rsync  version 3.1.2  protocol version 31
...

```





### 第二步：设置主服务器免密登录从服务器

在主服务器上操作

```shell
# 一直回车
[root@localhost ~]# ssh-keygen

# 发送公钥给从服务器,输入yes回车好，输入从服务器的root密码
[root@localhost ~]# ssh-copy-id root@192.168.0.16

# 尝试登录从服务器，查看是否已免密码登录
[root@localhost ~]# ssh root@192.168.0.16
[root@localhost ~]# hostname -I
192.168.0.16 

```





### 第三步：在主服务器安装inotify

inotify的包是叫inotify-tools，且需要epel源，安装好之后会新增两条命令：`inotifywait`、`inotifywatch`

```shell
[root@localhost ~]# yum insall -y inotify-tools
```



### 第四步：编写同步shell脚本

```shell
[root@localhost ~]# cat /Script/syn.sh 
#!/bin/bash
#inotiyf+rsync单向实时同步

MON_DIR=/data/
BACKUP_SERVER=192.168.0.16
BACKUP_DIR=/data/
BACKUP_PATH=$BACKUP_SERVER:$BACKUP_DIR

inotifywait -mqr -e create,move,delete,attrib,modify $MON_DIR |\
while read event;do
  rsync -a --delete $MON_DIR $BACKUP_PATH
done


```

**注意**  ：路径写法的区别！源目录后面加不加/也影响你的同步目录:

- 没加/，就是将目录本身同步过去；

- 目录加/，就是将目录里的内容同步过去！

### 第五步：运行

```shell
# 给脚本执行权限
[root@localhost Script]# chmod 755 /Script/syn.sh

# 后台运行脚本
[root@localhost Script]# nohup sh /Script/syn.sh &

# 用jobs -查看正在后台作业的脚本PID与状态
[root@localhost Script]# jobs -l
[1]+ 65690 运行中               nohup sh /Script/syn.sh &

# 设置开机自启动
[root@localhost backup]# echo "nohup sh /Script/syn.sh &" >>  /etc/rc.local

```



### 第六步：测试

在主服务器NFS目录里：新增文件、新增目录，并对目录和文件做修改删除、移动等操作。看从 - 备份服务器上的NFS目录是否还与主服务器NFS目录一致





##　搭建keepalive+NFS高可用

### 第一步：在两台NFS服务器上分别安装keepalived

```shell
[root@localhost ~]# yum install -y keepalived
```



### 第二步：分别配置keepalived

```shell
# 配置主NFS服务器的keepalived
[root@localhost ~]# hostname -I
192.168.0.15
[root@localhost ~]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived
global_defs {
router_id lb15
}

vrrp_script check_nfs {
script "/Script/check_nfs.sh"        --http服务启动脚本路劲
interval 2
}

vrrp_instance VI_1 {
state MASTER            --指定该节点为master节点
interface ens33            --这两行要注意换成自己网卡的名称
lvs_sync_daemon_interface ens33        --这两行要注意换成自己网卡的名称
virtual_router_id 51
priority 100            --这里要注意，主的要比备的大
advert_int 1
authentication {
auth_type PASS
auth_pass 1111
}
track_script {
check_nfs
}
virtual_ipaddress {
192.168.0.100/24            --虚拟IP地址（浮动IP）
}
}


```



```shell
# 配置从NFS服务器的keepalived
[root@localhost ~]# hostname -I
192.168.0.16 
[root@localhost ~]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived
global_defs {
router_id lb16
}

vrrp_script check_nfs {
script "/Script/check_nfs.sh"        --检测脚本
interval 2
}

vrrp_instance VI_1 {
state BACKUP            --指定该节点为master节点
interface ens33            --这两行要注意换成自己网卡的名称
lvs_sync_daemon_interface ens33        --这两行要注意换成自己网卡的名称
virtual_router_id 51
priority 50            --这里要注意，主的要比备的大
advert_int 1
authentication {
auth_type PASS
auth_pass 1111
}
track_script {
check_nfs
}
virtual_ipaddress {
192.168.0.100/24            --虚拟IP地址（浮动IP）  
}
}


```



### 第三步：配置shell脚本

用shell脚本实现如果监控的服务宕掉了，就把keepalived关闭，从而实现VIP漂移

```shell
# 在两台NFS服务器都配置上/Srcipt/check_nfs.sh脚本
[root@localhost ~]# cat /Script/check_nfs.sh 
#!/bin/bash
#auto check nfs process
NUM=$(netstat -ltp|grep nfs |wc -l)
if [[ $NUM -eq 0 ]];then
        systemctl stop keepalived
fi
```



### 第三步： 启动keepalived服务

```shell
# 在两台服务器上启动keepalived
[root@localhost backup]# systemctl restart keepalived
[root@localhost backup]# systemctl enable keepalived
```



### 第四步：测试

1.  在主NFS上会看到多了个ip地址，这就是VIP

```shell
[root@localhost ~]# hostname -I
192.168.0.15 192.168.0.100 

```

2. 停掉主NFS服务的nfs服务，模拟主NFS服务的nfs出现故障，看VIP是否有跳过从NFS服务器



## 自动挂载到web服务器

对于nfs的远程挂载，下面命令经常要做，有什么方法可以简化此操作`# mount -t nfs 192.168.0.16:/data /mnt/nfs2`

1. /etc/fstab

2. /etc/rc.local

3. 做别名

4. autofs

   

   --上面四种方式，前两种不建议用，因为nfs的挂载有个问题，你做了开机自动挂载我的nfs共享，如果我的nfs服务器关机了，而你没有去umount我的挂载，那么你的挂载目录会卡死（造成df -h等命令都用不了),这里建议用第四种

```shell
# 使用autofs前先取消之前对NFS的挂载
[root@localhost mnt]# yum install -y autofs
[root@localhost mnt]# vim /etc/auto.master
#在最后加一行
/mnt	/etc/auto.nfs
[root@localhost mnt]# vim /etc/auto.nfs
nfs1	-rw	192.168.0.15:/data
[root@localhost mnt]# systemctl restart autofs
[root@localhost mnt]# systemctl enable autofs

# 用绝对路径访问时，就会自动挂载NFS，5分钟内没有操作就会取消挂载
[root@localhost mnt]# ls /mnt/nfs1
1


```







## 故障

### （1）客户端——ls: 无法访问/mnt/nfs1: 失效文件句柄

描述：NFS服务器重启后，客户端执行df没有挂载nfs共享目录了。但是访问原来的挂载目录会报错，如下：

```shell
[root@localhost mnt]# ls /mnt/nfs1
ls: 无法访问/mnt/nfs1: 失效文件句柄
```



分析：

```shell
# 查看系统里是否还有之前的挂载信息
[root@localhost mnt]#  cat /proc/mounts |grep /mnt/nfs1
192.168.0.100:/data /mnt/nfs1 nfs4 rw,relatime,vers=4.1,rsize=65536,wsize=65536,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=192.168.0.13,local_lock=none,addr=192.168.0.100 0 0
```

由于不是正常取消挂载，所以系统没能完全取消挂载

解决：再取消挂载一次就行了

```shell
[root@localhost mnt]#  umount 192.168.0.100:/data
```



### （2）keepalived把VIP分配给从服务器后，客户端也会出现故障（1）无法自动挂载

原因：因为客户端没有在VIP飘从NFS服务器后，没有取消挂载。导致没能自动重新挂载。

解决办法：

（1）VIP发生漂移时，发出警报，运维尽快在客户端手动取消挂载

（2）在客户端写脚本检测，出现此报错时，自动取消挂载



### （3）rsync的实时同步不限制其网速的话，NFS主服务器写入压力太大时，就会导致写入十分缓慢

解决：给rsync加bwlimit=1000，限制rsync传输速度，单位是Bytes/s



### （4）脑裂问题

脑裂描述：由于主从服务器之间的网络出现了问题，例如访问被拒绝、延迟高等等，导致主从之间都获取不到对方的心跳信息，就都启用了VIP，导致用户

