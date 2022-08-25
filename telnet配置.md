版权声明：本文为CSDN博主「挖坑埋你」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/liupeifeng3514/article/details/79686740

本文笔记参考了以上博主

Telnet协议是TCP/IP协议族中的一员，是Internet远程登陆服务的标准协议和主要方式。它为用户提供了在本地计算机上完成远程主机工作的能力。在终端使用者的电脑上使用telnet程序，用它连接到服务器。终端使用者可以在telnet程序中输入命令，这些命令会在服务器上运行，就像直接在服务器的控制台上输入一样。可以在本地就能控制服务器。要开始一个telnet会话，必须输入用户名和密码来登录服务器。Telnet是常用的远程控制Web服务器的方法。

　　但是，telnet因为采用明文传送报文，安全性不好，很多Linux服务器都不开放telnet服务，而改用更安全的ssh方式了。但仍然有很多别的系统可能采用了telnet方式来提供远程登录，因此弄清楚telnet客户端的使用方式仍是很有必要的。
　　telnet命令还可做别的用途，比如确定远程服务的状态，比如确定远程服务器的某个端口是否能访问。

使用Telnet协议进行远程登录时需要满足以下条件：在本地计算机上必须装有包含Telnet协议的客户程序；必须知道远程主机的Ip地址或域名；必须知道登录标识与口令。

Telnet远程登录服务分为以下4个过程：

1）本地与远程主机建立连接。该过程实际上是建立一个TCP连接，用户必须知道远程主机的Ip地址或域名；

2）将本地终端上输入的用户名和口令及以后输入的任何命令或字符以NVT（Net Virtual Terminal）格式传送到远程主机。该过程实际上是从本地主机向远程主机发送一个IP数据包；

3）将远程主机输出的NVT格式的数据转化为本地所接受的格式送回本地终端，包括输入命令回显和命令执行结果；

4）最后，本地终端对远程主机进行撤消连接。该过程是撤销一个TCP连接。

#### 一、安装telnet

##### 1、检测telnet-server

rpm -qa telnet-server

(telnet-server.rpm默认没安装，输入rpm -qa telnet-server无输出则表示没有安装)

以下图为已安装

![image-20220825105821083](C:\Users\XL\AppData\Roaming\Typora\typora-user-images\image-20220825105821083.png)

##### 2、安装telnet-server

yum install telnet-server

##### 3.检测telnet的rpm包

rpm -qa telnet

![image-20220825105952553](C:\Users\XL\AppData\Roaming\Typora\typora-user-images\image-20220825105952553.png)

##### 4.安装telnet

yum install telnet

#### 二、安装xinetd服务

由于telnet服务也是由xinetd守护的，所以安装完telnet-server，要启动telnet服务就必须重新启动xinetd 。

xinetd即extended internet daemon，xinetd是新一代的网络守护进程服务程序，又叫超级Internet服务器。经常用来管理多种轻量级Internet服务。telnet服务就是通过xinetd服务来管理的，所以在安装telnet服务之前需要先安装xinetd服务。
1.查看xinetd服务

rpm -qa | grep xinetd

![image-20220825110335985](C:\Users\XL\AppData\Roaming\Typora\typora-user-images\image-20220825110335985.png)

2.安装xinetd包

yum install -y xinetd

3.重启xinetd服务

service xinetd restart

4.查看xinetd状态

service xinetd status

![image-20220825110528774](C:\Users\XL\AppData\Roaming\Typora\typora-user-images\image-20220825110528774.png)

#### 三、注意点及错误的解决

##### 1、若xinetd未安装，则安装。

以上是linux安装telnet命令的全部过程，可能在安装过程中遇到问题，
在启动 xinetd.service 时提示
Redirecting to /bin/systemctl restart xinetd.service
Failed to issue method call: Unit xinetd.service failed to load: No such file or directory.
说明系统没有安装 xinetd,需要使用 yum -y instal xinetd.service进行服务的安装
在启动xinetd.service时出现:
Redirecting to /bin/systemctl restart xinetd.service
可能启动的命令是systemctl restart xinetd.service



##### 2、更改配置文件/etc/xinetd.d/telnet

若此文件不存在，则创建这个文件。将其中disable=yes改为disable=no或注释掉。即改为如下：

`#default: on`

`#description: The telnet server serves telnet sessions; it uses \`

`#unencrypted username/password pairs for authentication.`

`service telnet`
`{`
        `disable = no`
        `flags           = REUSE`
        `socket_type     = stream`
        `wait            = no`
        `user            = root`
        `server          = /usr/sbin/in.telnetd`
        `log_on_failure  += USERID`

`}`

![image-20220825110956616](C:\Users\XL\AppData\Roaming\Typora\typora-user-images\image-20220825110956616.png)

##### 3.使用root登录

telnet 默认的情况之下不允许 root 以 telnet 进入 Linux 主机，在普通用户telnet进入系统之后，在切换到root用户就可以使用root用户了。

###### 1.修改login文件

[redhat](https://so.csdn.net/so/search?q=redhat&spm=1001.2101.3001.7020)中对于远程登录的限制体现在/etc/pam.d/login 文件中，如果把限制的内容注销掉，那么限制将不起作用。

`#%PAM-1.0`
`auth [user_unknown=ignore success=ok ignore=ignore default=bad] pam_securetty.so`
`auth       include      system-auth`
`#account    required     pam_nologin.so`
`account    include      system-auth`
`password   include      system-auth`

`#pam_selinux.so close should be the first session rule`

`session    required     pam_selinux.so close`
`session    include      system-auth`
`session    required     pam_loginuid.so`
`session    optional     pam_console.so`

`#pam_selinux.so open should only be followed by sessions to be executed in the user context`

`session    required     pam_selinux.so open`
`session    optional     pam_keyinit.so force revoke`

将文件中的 pam_securetty.so行，加上“#”注释掉；

如果成功telnet则证明修改成功，如果无法telnet输入

tail /var/log/secure

可以看到在这里被拒绝了,根据提示在第三种方法中加入pts/1

![image-20220825112456345](C:\Users\XL\AppData\Roaming\Typora\typora-user-images\image-20220825112456345.png)

如果非要使用root登录，可以使用修改securetty文件。

###### 2.移除securetty文件

验证规则设置在/etc/security文件中，该文件定义root用户只能在tty1-tty6的终端上记录，删除该文件或者将其改名即可避开验证规则实现root用户[远程登录](https://so.csdn.net/so/search?q=远程登录&spm=1001.2101.3001.7020)。

mv /etc/securetty /etc/securetty.bak

###### 3.修改securetty文件

vi /etc/securetty，在其中加入pts/1 - pts/11

![image-20220825111136078](C:\Users\XL\AppData\Roaming\Typora\typora-user-images\image-20220825111136078.png)

###### 4、修改telnet端口

修改文件/etc/services将文件中

![image-20220825120600832](D:\笔记\note\telnet配置\image-20220825120600832.png)

修改为

![image-20220825120945530](D:\笔记\note\telnet配置\image-20220825120945530.png)

重启telnet服务即可。