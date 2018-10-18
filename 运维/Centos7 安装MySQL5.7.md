# **Centos7 安装MySQL5.7**

原转：https://www.cnblogs.com/mujingyu/p/7689116.html 做了修改

参考：

https://www.cnblogs.com/wzg123/p/6723338.html

https://blog.csdn.net/yabingshi_tech/article/details/51051922

https://blog.csdn.net/qq_32331073/article/details/76229420

前言：**经过一天半的折腾，终于把 mysql 5.7.17 版本安装上了 centos 7 系统上，把能参考的博客几乎都看了一遍，终于发现这些细节问题，然而翻了无数的文章，基本上都没有提到这些，所以小生尽量把这些细节写下来，一方面是供初学者们参考，另一方面也是对自己花这么长时间的摸索的一个总结，如有不足之处欢迎各路高手指正。

**一、安装前的检查**

　　**1.1 检查 linux 系统版本**

　　　　[root@localhost ~]# cat /etc/system-release

　　　　　　说明：小生的版本为 linux 64位：CentOS Linux release 7.4.1708 (Core)

　　**1.2 检查是否安装了 mysql**

　　　　[root@localhost ~]# rpm -qa | grep mysql

　　　　　　若存在 mysql 安装文件，则会显示 mysql安装的版本信息

　　　　　　　　如：mysql-connector-odbc-5.2.5-6.el7.x86_64

　　　　　　卸载已安装的MySQL，卸载mysql命令，如下：

　　　　　　　　[root@localhost ~]# rpm -e --nodeps mysql-connector-odbc-5.2.5-6.el7.x86_64

　　　　　　将/var/lib/mysql文件夹下的所有文件都删除干净。

> 　　　　**细节注意：**
>
> 　　　　　　检查一下系统是否存在 mariadb 数据库，如果有，一定要卸载掉，否则可能与 mysql 产生冲突。
>
> 　　　　　　小生的系统安装模式的是最小安装，所以没有这个数据库。
>
> 　　　　　　检查是否安装了 mariadb：[root@localhost ~]# rpm -qa | grep mariadb
>
> 　　　　　　如果有就使劲卸载干净：
>
> 　　　　　　　　systemctl stop mariadb
> 　　　　　　　　rpm -qa | grep mariadb
> 　　　　　　　　rpm -e --nodeps mariadb-5.5.52-1.el7.x86_64
> 　　　　　　　　rpm -e --nodeps mariadb-server-5.5.52-1.el7.x86_64
> 　　　　　　　　rpm -e --nodeps mariadb-libs-5.5.52-1.el7.x86_64

　　**1.3 系统内存检查**

　　　　检查一下 linux 系统的虚拟内存大小，如果内存不足 1G，启动 mysql 的时候可能会产生下面这个错误提示：　　

　　　　　　Starting mysqld (via systemctl): Job for mysqld.service failed because the control process exited with error code.

　　　　　　See "systemctl status mysqld.service" and "journalctl -xe" for details.[FAILED]

　　　　小生起初安装的时候使用的是虚拟机自动分区，内存设置的是 1G，结果在这上面话费了大量的精力和时间去调试始终也启动没成功。

　　　　最后在一位 [远走的兔子](http://blog.csdn.net/u014182411) 博客里找到了这个问题的答案：　　　　

> 　　通过lnmp安装mysql之后，显示mysql is not running。出错如下：
>
> 　　　　Starting MySQL..The server quit without updating PID file (/usr/local/mysql/var/localhost.localdomain.pid)
>
> 　　网上各种找解决方案，大部分都是什么文件权限、mysql 日志太大，重装等问题。一一试了解决方案，然并卵……
>
> 　　最后，打开看了lnmp的官方网站的安装教程 https://lnmp.org/install.html，才知道原来是因为自己安装时选择的mysql版本是5.6，而**安装5.6以及以上版本的mysql需要服务器的内存至少在1G以上**。而我是在VPS上装的。内存在256MB，不出错才有问题。瞬间心塞。
>
> 　　最后先卸载 lnmp，再安装。安装完全可以参照官方教程。

**二、从 mysql 官网下载并上传 mysql安装包**

　　**2.1 下载 mysql 安装包**

　　　　小生使用的是 mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz

　　　　![img](https://images2017.cnblogs.com/blog/1070942/201710/1070942-20171019203214474-124178132.png)

　　**2.2 上传安装文件到 linux 系统**

　　　　使用 ftp 上传至 linux 系统中，小生上传至 /var/ftp/pub 文件目录下

> 　　　　细节注意：
>
> 　　　　　　出于安全问题，建议使用 md5sum 命令核对一下文件源：[root@localhost pub]# md5sum mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz

**三、安装 mysql**

　　**3.1 解压安装包，并移动至 home 目录下**

　　　　解压 mysql 的 gz 安装包： [root@localhost pub]# tar -zvxf mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz

　　　　/usr/local 目录下创建文件夹存 mysql：[root@localhost pub]# mkdir /usr/local/msyql

　　　　将文件移动到 /usr/local 目录下，并重命名文件夹：[root@localhost pub]# mv mysql-5.7.17-linux-glibc2.5-x86_64/* /usr/lcoal/mysql

　　**3.2 添加系统用户**

　　　　添加 mysql 组和 mysql 用户：

　　　　　　添加 mysql 组：[root@localhost ~]# groupadd mysql

　　　　　　添加 mysql 用户：[root@localhost ~]# useradd -r -g mysql mysql

　　　　　　扩展：

　　　　　　　　查看是否存在 mysql 组：[root@localhost ~]# more /etc/roup | grep mysql

　　　　　　　　查看 msyql 属于哪个组：[root@localhost ~]# groups mysql

　　　　　　　　查看当前活跃的用户列表：[root@localhost ~]# w

　　**3.3 检查是否安装了 libaio**

　　　　[root@localhost pub]# rpm -qa | grep libaio

　　　　若没有则安装

　　　　　　版本检查：[root@localhost pub]# yum search libaio

　　　　　　安装：[root@localhost pub]# yum -y install libaio

　　**3.3 安装 mysql**

　　　　进入安装 mysql 软件目录：[root@localhost ~]# cd /usr/local/mysql/

　　　　安装配置文件：[root@localhost mysql]# cp ./support-files/my-default.cnf /etc/my.cnf（提示是否覆盖，输入“ y ”同意）

　　　　　　修改被覆盖后的 my.cnf：[root@localhost mysql]# vim /etc/my.cnf

> [mysql]  
>
> \# 设置mysql客户端默认字符集  
>
> default-character-set=utf8   
>
> socket=/var/lib/mysql/mysql.sock  
>
> [mysqld]  
>
> \#skip-name-resolve  
>
> \#设置3306端口  
>
> port = 3306   
>
> socket=/var/lib/mysql/mysql.sock  
>
> \# 设置mysql的安装目录  
>
> basedir=/usr/local/mysql  
>
> \# 设置mysql数据库的数据的存放目录  
>
> datadir=/usr/local/mysql/data  
>
> \# 允许最大连接数  
>
> max_connections=200  
>
> \# 服务端使用的字符集默认为8比特编码的latin1字符集  
>
> character-set-server=utf8  
>
> \# 创建新表时将使用的默认存储引擎  
>
> default-storage-engine=INNODB  
>
> \#lower_case_table_name=1  
>
> max_allowed_packet=16M  
>
> [client]
> default-character-set=utf8
> socket=/var/lib/mysql/mysql.sock
>
> [mysql]
> default-character-set=utf8
> socket=/var/lib/mysql/mysql.sock



　　　　创建 data 文件夹：[root@localhost mysql]# mkdir ./data

　　　　修改当前目录拥有者为 mysql 用户：[root@localhost mysql]# chown -R mysql:mysql ./

　　　　初始化 mysqld：[root@localhost mysql]# ./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/

​               创建运行日志目录 :[root@localhost mysql]# mkdir /var/lib/mysql

​	     修改当前目录拥有者为 mysql 用户：`chown -R mysql:mysql /var/run/mysql/`  

　　　　![img](https://images2017.cnblogs.com/blog/1070942/201710/1070942-20171019210451318-1634435762.png)

　**四、配置 mysql**

　　**4.1 设置开机启动** 

　　　　a. 复制启动脚本到资源目录：[root@localhost mysql]# cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld

　　　　b. 增加 mysqld 服务控制脚本执行权限：[root@localhost mysql]# chmod +x /etc/rc.d/init.d/mysqld

　　　　c. 将 mysqld 服务加入到系统服务：[root@localhost mysql]# chkconfig --add mysqld

　　　　d. 检查mysqld服务是否已经生效：[root@localhost mysql]# chkconfig --list mysqld

　　　　　　命令输出类似下面的结果：

　　　　　　　　mysqld 0:off 1:off 2:on 3:on 4:on 5:on 6:off 

　　　　　　表明mysqld服务已经生效，在2、3、4、5运行级别随系统启动而自动启动，以后可以使用 service 命令控制 mysql 的启动和停止。

　　　　　　查看启动项：chkconfig --list | grep -i mysql

　　　　　　删除启动项：chkconfig --del mysql

　　　　e. 启动 mysqld：[root@localhost mysql]# service mysqld start

　　**4.2 环境变量配置**

　　　　将mysql的bin目录加入PATH环境变量，编辑 /etc/profile文件：[root@localhost mysql]# vim /etc/profile

> PATH = $PATH:/usr/local/mysql/bin 
>
> export PATH 

　　　　执行命令使其生效：[root@localhost mysql]# source /etc/profile

　　　　用 export 命令查看PATH值：[root@localhost mysql]# echo $PATH

**五、登录 mysql**

　　**5.1 测试登录**

　　　　登录 mysql：[root@localhost mysql]# mysql -uroot -p（登录密码为初始化的时候显示的临时密码）

　　　　初次登录需要设置密码才能进行后续的数据库操作：SET PASSWORD = PASSWORD('123456');（密码设置为了123456） 

　　　　修改密码为 password：update user set authentication_string=PASSWORD('password') where User='root';

　　**5.2 防火墙端口偶设置，便于远程访问**

```
[root@localhost ~]$ firewall-cmd --zone=public --add-port=3306/tcp --permanent
[root@localhost ~]$ firewall-cmd --reload
```



> 　　**开启防火墙mysql3306端口的外部访问**
>
> 　　CentOS升级到7之后，使用firewalld代替了原来的iptables。下面记录如何使用firewalld开放Linux端口
>
> 　　--zone : 作用域，网络区域定义了网络连接的可信等级。
>
> 　　　　这是一个一对多的关系，这意味着一次连接可以仅仅是一个区域的一部分，而一个区域可以用于很多连接
>
> 　　--add-port : 添加端口与通信协议，格式为：端口/通讯协议，协议是tcp 或 udp
>
> 　　--permanent : 永久生效，没有此参数系统重启后端口访问失效

　　5.3 使用 SQLyog 远程连接出现不允许连接问题：

　　　　首先使用 dos 窗口 ping 一下 linux，排除网络连通问题，其次使用 SQLyog 连接测试一下。

　　　　解决方法：登录 linux mysql 在用户管理表新增用户帐号

　　　　　　mysql> use msyql 

　　　　　　mysql> create user 'user-name'@'ip-address' identified by 'password';（红色标记为需要修改的地方）

　　　　其他方案：

　　　　　　授权root用户可以进行远程连接，注意替换以下代码中的“password”为 root 用户真正的密码，

　　　　　　另外请注意如果你的root用户设置的是弱口令，那么非常不建议你这么干！：　　　

```
mysql> grant all privileges on . to root@"%" identified by "password" with grant option;
mysql> flush privileges;
```

