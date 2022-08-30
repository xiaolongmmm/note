

本笔记参考CSDN博主「也也来噜」的原创文章并进行实操。
原文链接：https://blog.csdn.net/qq_48111800/article/details/124693250在Linux下安装WEB服务器

每次搭建一个服务器之前，比如MySQL、DNS、WEB等首先要挂载磁盘目录文件
挂载就是当要使用某个设备时（例如光盘或软盘），必须先将它们对应放到 Linux 系统中的某个目录上。其中对应的目录就叫作挂载点。只有经过操作之后，用户或程序才能访问到这些设备。这个操作过程就叫作文件系统的挂载。这里/dev/sr0是软盘，/mnt/cdrom是挂载点

#### 一、挂载

mount /dev/sr0 /mnt/cdrom/

如果提示cdrom不是一个挂载点，大概率原因是没有该目录，去创建一个目录即可。

![image-20220830161022383](D:\笔记\note\WEB服务器配置\image-20220830161022383.png)

#### 二、搭建Web服务器

##### 1.查找httpd服务

`yum search httpd`

![image-20220830161416447](D:\笔记\note\WEB服务器配置\image-20220830161416447.png)

##### 2.安装httpd服务

`yum install httpd.x86_64 -y`

##### 3.启动httpd服务

启动 `systemctl start httpd`

查看状态 `systemctl status httpd`

开机自启 `systemctl enable httpd`

关闭 `systemctl stop httpd`

##### 4.防火墙设置

① `semanage port -l | grep -w http_port_t`  *查看默认允许的端口*

② `firewall-cmd --permanent --add-port=80/tcp` 默认是开启80端口

③ `firewall-cmd --permanent --add-service=http`

④ `systemctl disable firewalld` 永久关闭防火墙

⑤ `Permission denied: make_sock: could not bind to address 0.0.0.0:843` 如果遇到该种情况，大概率是因为SELinux

直接修改/etc/selinux/config找到SELINUX=enforcing 修改为**SELINUX=disable**，然后reboot即可。

很多地方提到的都是setenforce 0，但最开始这样设置在开机的时候服务报错了。

##### 5.访问http

![image-20220830162632670](D:\笔记\note\WEB服务器配置\image-20220830162632670.png)

##### 6.编辑个人网页

① `cd /etc/httpd/conf`

​    `cp httpd.conf httpd.conf.bak`

​    `vi httpd.conf`

② :set nu 编辑时可以查看行数

③ 修改配置，所有访问授权

![image-20220830163009672](D:\笔记\note\WEB服务器配置\image-20220830163009672.png)

④ 在httpd.conf这个配置文件中配置浏览器访问的网页根目录：可以自定义更改

![image-20220830163112344](D:\笔记\note\WEB服务器配置\image-20220830163112344.png)

⑤ 开放访问/var/www目录的权限

![image-20220830163208873](D:\笔记\note\WEB服务器配置\image-20220830163208873.png)

⑥ `Options Indexes FollowSymLinks`语句：如果该虚拟目录下没有 index.html,浏览器会显示该虚拟目录的目录结构,列出该虚拟目录下的文件和子目录。

![image-20220830163258487](D:\笔记\note\WEB服务器配置\image-20220830163258487.png)



⑦ `AllowOverride None`: 完全忽略`.htaccess` 文件(超文本文件)

![image-20220830163352793](D:\笔记\note\WEB服务器配置\image-20220830163352793.png)

⑧ 设置浏览器默认访问的网页为index.html

![image-20220830163422351](D:\笔记\note\WEB服务器配置\image-20220830163422351.png)

⑨测试

`cd /var/www/html`

`mkdir stu`

`cd stu`

`echo "this student score web">index.html`

![image-20220830163617908](D:\笔记\note\WEB服务器配置\image-20220830163617908.png)

⑩测试`Options Indexes FollowSymLinks`属性

`rm -rf index.html`

`touch 1.txt 2.txt`

![image-20220830163730480](D:\笔记\note\WEB服务器配置\image-20220830163730480.png)

#### 三、基于虚拟主机IP搭建

1.网卡配置，在网卡中新添加两个ip

`IPADDR1=192.168.182.100` 

`IPADDR2=192.168.182.200`

![image-20220830164108919](D:\笔记\note\WEB服务器配置\image-20220830164108919.png)

2.配置虚拟主机IP地址的网页访问主目录和网页index.html文件

`cd /var/www/`

`mkdir 100 200`

`echo 192.168.182.100 test website > 100/index.html`

`echo 192.168.182.200 test website > 200/index.html`

3.配置虚拟主机ip地址的httpd配置文件
回到配置文件的目录下面，创建一个vhost目录，里面专门存放虚拟IP地址的配置文件，然后在httpd服务的主配置文件中引用这个目录

`cd /etc/httpd`

`mkdir vhost`

`cd vhost/`

 `touch 100.conf 200.conf`

4.配置虚拟主机IP的配置文件

![image-20220830164702725](D:\笔记\note\WEB服务器配置\image-20220830164702725.png)

5.配置`/etc/httpd/conf/httpd.conf`主配置文件引用`vhost`目录里的虚拟主机IP的`.conf`配置文件

快捷键`Shift+g`可以直接到达文件最后一行

在主配置文件的末尾加上

`IncludeOptional vhost*/\*.conf*`

![image-20220830164903320](D:\笔记\note\WEB服务器配置\image-20220830164903320.png)

6.重启httpd服务，在物理机浏览器查看虚拟主机IP的网页配置

![image-20220830164930370](D:\笔记\note\WEB服务器配置\image-20220830164930370.png)

![image-20220830164940424](D:\笔记\note\WEB服务器配置\image-20220830164940424.png)

#### 四、基于端口号搭建

1.首先创建网页文件html

`cd /var/www/`

`mkdir 8081 8082`

`echo 8081 port test web > 8081/index.html`

`echo 8082 port test web > 8082/index.html`

2.然后在vhost目录下配置这两个虚拟端口的配置文件

`cd /etc/httpd/vhost`

`cp  100.conf 8081.conf`

`cp  100.conf 8082.conf`

![image-20220830165427501](D:\笔记\note\WEB服务器配置\image-20220830165427501.png)

3.配置`/etc/httpd/conf/httpd.conf`主配置文件引用`vhost`目录里的虚拟主机IP的`.conf`配置文件配置端口号

![image-20220830165539872](D:\笔记\note\WEB服务器配置\image-20220830165539872.png)

(在这里还需要添加一句话，`ServerName localhost:80`)否则可能重启失败

![image-20220830165623121](D:\笔记\note\WEB服务器配置\image-20220830165623121.png)

4.配置完成保存退出，防火墙运行端口8081和8082通过，重启httpd服务，测试虚拟端口是否可用

`firewall-cmd --permanent --add-port=8081/tcp`

`firewall-cmd --permanent --add-port=8082/tcp`

`firewall-cmd --reload`

`setenforce 0`

`getenforce`

`systemctl restart httpd`

5.测试

![image-20220830165815271](D:\笔记\note\WEB服务器配置\image-20220830165815271.png)

![image-20220830165824627](D:\笔记\note\WEB服务器配置\image-20220830165824627.png)

#### 五、个人Web站点搭建

1.conf.d文件配置

`cd /etc/httpd/conf.d`

`cp userdir.conf  userdir.conf.bak`

`vi userdir.conf`

![image-20220830170355855](D:\笔记\note\WEB服务器配置\image-20220830170355855.png)

2.配置完成，重启httpd服务

3.新增用户user1

`useradd user1`

`passwd user1`

4.创建个人站点网页文件

`su - user1`

`cd ..`

`chmod -Rf 711 user1`

`mkdir -p user1/public_html/`

`echo user1 test website > user1/public_html/index.html`

5.测试

![image-20220830170647370](D:\笔记\note\WEB服务器配置\image-20220830170647370.png)

