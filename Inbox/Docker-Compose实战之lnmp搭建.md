提前：Docker环境
1.下载docker-compose
2.为nginx准备配置文件nginx.conf，与测试文件index.html、test.html、mysql.html
3.编写docker-compose文件，docker-compose.yml
![docker-compose dir](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/docker%E5%87%86%E5%A4%87.png)
4.执行docker-compose up -d，执行成功则通过浏览器访问nginx的测试文件
![](https://images-1300072815.cos.ap-shenzhen-fsi.myqcloud.com/docker-compose-su.png)

5.如有报错，则使用docker logs id，通过日志排错。然后先用docker-compose down停止容器后，再启动