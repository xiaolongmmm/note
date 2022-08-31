本文参考CSDN博主「也也来噜」的原创文章做自己的笔记以及实操
原文链接：https://blog.csdn.net/qq_48111800/article/details/124597195

Linux下的DHCP服务器搭建实战

DHCP是Dynamic Host [Configuration](https://so.csdn.net/so/search?q=Configuration&spm=1001.2101.3001.7020) Protocol的缩写，即动态主机配置协议。DHCP是一个很重要的局域网的网络协议，使用UDP协议工作，主要有以下用途：

1、为内部网络或网络服务供应商自动分配IP地址；

2、为用户或者内部网络管理员作为对所有计算机作中央管理的手段；

3、为内部网络用户接受IP租约。

#### 一、搭建服务

1.安装DHCP服务

`yum install dhcp`

`yum install dhcp.x86_64`

2.使用rpm命令查看dhcpd的安装包

`rpm -qa | grep dhcp`

3.挂载

`mount /dev/cdrom /mnt/cdrom/`

4.查找dhcp配置文件的示例文件

`find / -name dhcp*`

![image-20220831110009216](D:\笔记\note\DHCP服务器配置\image-20220831110009216.png)

5.将示例文件的内容复制到配置文件`/etc/dhcp/dhcpd.conf`中

`cd /usr/share/doc/dhcp-4.2.5/`

`cp -a dhcpd.conf.example /etc/dhcp/dhcpd.conf`

`cd /etc/dhcp`

6.开始配置dhcp的配置文件

`vi dhcpd.conf`

去掉`ddns-update-style none`前的注释符，意思是禁止动态更新

![image-20220831110205301](D:\笔记\note\DHCP服务器配置\image-20220831110205301.png)

配置要分配的网段的范围
语句① 分配网段是192.168.182.0 子网掩码是24
语句② 分配的地址范围是192.168.182.220 ~ 192.168.182.230
语句③ 设置这个分配下去的地址范围的主机的DNS服务器的IP地址 192.168.182.128
语句④ 设置DNS服务器的域名为dns.mysqlz.com
语句⑤ 设置这个分配网段的网关地址 192.168.182.2
语句⑥ 设置这个网段的的广播地址 192.168.182.255
语句⑦ 设置默认的租约时间 为600秒
语句⑧ 设置最大的租约时间 为7200秒

![image-20220831111601833](D:\笔记\note\DHCP服务器配置\image-20220831111601833.png)



7.配置完成之后要重启DHCP服务

8.配置虚拟网络编辑器

![image-20220831112546999](D:\笔记\note\DHCP服务器配置\image-20220831112546999.png)

9.使用另外一台虚拟机或者物理机测试dhcp

![image-20220831112902235](D:\笔记\note\DHCP服务器配置\image-20220831112902235.png)

![image-20220831122604935](D:\笔记\note\DHCP服务器配置\image-20220831122604935.png)

