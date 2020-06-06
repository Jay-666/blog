## 1、下载



```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
```

也可以在浏览器去[maven官网](http://maven.apache.org/download.cgi)下载需要的版本，这里安装的是二进制包，所以选择“-bin.tar.gz”结尾的包

## 2、解压

```shell
tar -xf apache-maven-3.6.3-bin.tar.gz -C /usr/local/
mv /usr/local/apache-maven-3.6.3/ maven3.6

```

## 3、加入环境变量

在/etc/profile文件最下方加入新的一行`export PATH=$PATH:/usr/local/maven3.6/bin`  

添加完后，执行`source /etc/profile`,让配置生效

验证：

执行` which mvn`
显示/usr/local/maven3.6/bin/mvn就说明配置成功了

## 4、JAVA环境

maven需要Java环境，需要系统安装有jdk，并且在系统中配置了JAVA_HOME。