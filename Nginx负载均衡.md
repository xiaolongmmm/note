Linux配置nginx负载均衡

​		企业在解决高并发问题时，一般有两个方向的处理策略，软件、硬件，硬件上添加负载均衡器分发大量请求，软件上可在高并发瓶颈处：数据库+web服务器两处添加解决方案，其中web服务器前面一层最常用的的添加负载方案就是使用nginx实现负载均衡。

转发功能：

​		按照一定的算法【权重、轮询】，将客户端请求转发到不同应用服务器上，减轻单个服务器压力，提高系统并发量。

故障移除：

​		通过心跳检测的方式，判断应用服务器当前是否可以正常工作，如果服务器期宕掉，自动将请求发送到其他应用服务器。

回复添加：

​		如检测到发生故障的应用服务器恢复工作，自动将其添加到处理用户请求队伍中。

#### 一、部署Nginx

1.安装依赖包

```
	yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

2.下载并且解压安装包
	

```
	cd /usr/local
	mkdir nginx
	cd nginx
	wget http://nginx.org/download/nginx-1.13.7.tar.gz
	tar -xvf nginx-1.13.7.tar.gz
```

3.安装nginx

```
	cd /usr/local/nginx
	cd nginx-1.13.7
	./configure --with-http_stub_status_module --with-http_ssl_module
	make
	make install
```

4.启动nginx服务

```
	cd /usr/local/nginx/sbin

	./nginx -c /usr/local/nginx/conf/nginx.conf
```

5.配置文件路径

```
	vi /usr/local/nginx/conf/nginx.conf
```

6.重启nginx

```
	cd /usr/local/nginx/sbin
	./nginx -s reload
```

7.查看进程(两种方法)

```
	① ps -ef | grep nginx
	② lsof -i tcp:(端口号：默认80)
```

8.杀死进程

```
	kill -9 (进程号/PID)
```

![image-20220915113415010](D:\笔记\note\Nginx负载均衡\image-20220915113415010.png)

9.访问（出现以下页面说明成功）

![image-20220915142724960](D:\笔记\note\Nginx负载均衡\image-20220915142724960.png)

#### 二、部署Apache

1.安装Apache，在一台服务器中同时部署三个不同端口的tomcat

![image-20220915150631151](D:\笔记\note\Nginx负载均衡\image-20220915150631151.png)

2.修改配置文件

```
	vim /usr/local/tomcat/apache-tomcat-1/conf/server.xml
```

![image-20220915151135730](D:\笔记\note\Nginx负载均衡\image-20220915151135730.png)

![image-20220915151208147](D:\笔记\note\Nginx负载均衡\image-20220915151208147.png)

![image-20220915151230575](D:\笔记\note\Nginx负载均衡\image-20220915151230575.png)

```
	vim /usr/local/tomcat/apache-tomcat-2/conf/server.xml
```

![image-20220915151455498](D:\笔记\note\Nginx负载均衡\image-20220915151455498.png)

![image-20220915151519179](D:\笔记\note\Nginx负载均衡\image-20220915151519179.png)

![image-20220915151536418](D:\笔记\note\Nginx负载均衡\image-20220915151536418.png)

```
	vim /usr/local/tomcat/apache-tomcat-3/conf/server.xml
```

![image-20220915151632250](D:\笔记\note\Nginx负载均衡\image-20220915151632250.png)

![image-20220915151650945](D:\笔记\note\Nginx负载均衡\image-20220915151650945.png)

![image-20220915151717657](D:\笔记\note\Nginx负载均衡\image-20220915151717657.png)

3.修改标识

```
	vim /usr/local/tomcat/apache-tomcat-1/webapps/ROOT/index.jsp
```

![image-20220915152207778](D:\笔记\note\Nginx负载均衡\image-20220915152207778.png)

其它同上

4.启动apache

```
	cd /usr/local/tomcat/apache-tomcat-1/bin/
	./startup.sh
	cd /usr/local/tomcat/apache-tomcat-2/bin/
	./startup.sh
	cd /usr/local/tomcat/apache-tomcat-3/bin/
	./startup.sh
```

5.测试页面

以下三个页面代表成功

![image-20220915152534328](D:\笔记\note\Nginx负载均衡\image-20220915152534328.png)

![image-20220915152554552](D:\笔记\note\Nginx负载均衡\image-20220915152554552.png)

![image-20220915152618977](D:\笔记\note\Nginx负载均衡\image-20220915152618977.png)

#### 三、Nginx实现负载均衡

1.配置nginx文件

```
	vim /usr/local/nginx/conf/nginx.conf
```

![image-20220915153333354](D:\笔记\note\Nginx负载均衡\image-20220915153333354.png)

![image-20220915153513794](D:\笔记\note\Nginx负载均衡\image-20220915153513794.png)

2.重启服务

```
	cd /usr/local/nginx/sbin

	./nginx -c /usr/local/nginx/conf/nginx.conf
```

3.测试输入nginx中ip

刷新几次为不同的网页负载成功

![image-20220915153814744](D:\笔记\note\Nginx负载均衡\image-20220915153814744.png)

![image-20220915153826255](D:\笔记\note\Nginx负载均衡\image-20220915153826255.png)

![image-20220915153836695](D:\笔记\note\Nginx负载均衡\image-20220915153836695.png)