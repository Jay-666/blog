### 问题

因为写防止DOS攻击的脚本，需要用date取现在的时间与nginx的access.log里的时间对比，但是access.log记录的月份是英文，date输出的格式却是中文。

### 解决思路

1. 改变系统默认字符集,但是中文会乱码
2. 或者用date -R，按照RFC 5322格式输出，例如: Tue, 05 May 2020 15:04:55 +0800 ，再通过awk组合成需要的格式

### 方法1：修改字符集

```shell
vim /etc/profile #在底部加上一行
export LANG="en_US.UTF-8"

source /etc/profile
```



### 方法2：RFC 5322格式输出

```shell
date -R|awk 'NR==1{print $2"/"$3"/"$4":"$5}'
#05/May/2020:21:00:25
```



> 参考: https://blog.csdn.net/qq_35663625/article/details/103678368