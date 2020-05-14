# CentOS 7安装MySQL5.7版本

##  1。 下载并安装mysql官方的yum源
- 使用root用户，在CentOS 7服务器的/yzy/soft路径下执行以下命令
```shell
su root
#在线安装wget工具
yum -y install wget
#在线安装上传下载工具
yum -y install lrzsz

cd /yzy/soft/
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```

> -i 指定输入文件
>
> -c 表示断点续传

## 2。安装mysql
```shell
yum -y install mysql57-community-release-el7-10.noarch.rpm
```
![](/mysql/mysql57yum.png)

- 安装mysql server
  这步可能会花些时间，需要在线下载，视网速而定；然后再安装；安装完成后就会覆盖掉之前的mariadb
```shell
yum -y install mysql-community-server
```
![](/mysql\mysqlserveryum.png)

## 3。设置mysql

### 3.1 mysql服务
- 首先启动MySQL服务
```shell
systemctl start mysqld.service
```
- 查看mysql启动状态
```
systemctl status mysqld.service
```
![](/mysql\mysqlstatus.png)

### 3.2 修改密码

- 此时MySQL已经开始正常运行，不过要登陆MySQL，还得先找出此时mysql的root用户的临时密码

  如下命令可以在日志文件中找出临时密码
```shell
grep "password" /var/log/mysqld.log
```
- 可以查看到我的临时密码为
 ![](/mysql\greppwd.png)

  > 注意：不同人的临时密码不一样，根据自己的实际情况而定==
```
fHy3Su:&REkh
```
- 使用临时密码，登陆mysql客户端
```mysql
mysql -uroot -p
```
- 设置密码策略为LOW，此策略只检查密码的长度
```mysql
set global validate_password_policy=LOW;
```
> 关键字“Query OK”表示，sql语句执行成功

- 设置密码最小长度

```mysql
set global validate_password_length=6;
```

![](/mysql\setpwdlength.png)

- 修改mysql的root用户，本地登陆的密码为123456

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```

- 开启mysql的远程连接权限

```mysql
grant all privileges  on  *.* to 'root'@'%' identified by '123456' with grant option;
flush privileges;
```

- 若不再需要使用mysql命令行，可以退出

```mysql
exit
```

## 4。mysql的卸载

==注意：mysql安装有问题的，才做此步骤==

- 上面我们在CentOS 7当中已经安装好了5.7版本的mysql服务；
- 如果以后我们不需要mysql了，或者mysql安装失败了需要重新安装，那么我们需要将mysql卸载掉
- ==使用root用户==

### 4.1 停止mysql服务

```shell
systemctl stop mysqld.service
```
### 4.2 列出已安装的mysql相关的包
- 有两种方式，都可以，任选其一
```shell
yum list installed mysql*
```
![](/mysql\yumlist.png)

```shell
rpm -qa | grep -i mysql
```
![](/mysql\rpmgrepmysql.png)

### 4.3 卸载mysql包

- 卸载rpm包，使用rpm -e --nodeps方式卸载，后边依次加入上图的①~⑥的包名，包名之间有空格

  > 注意：根据自己的实际情况，指定包名进行卸载==

```shell
rpm -e --nodeps mysql57-community-release-el7-10.noarch mysql-community-common-5.7.28-1.el7.x86_64 mysql-community-client-5.7.28-1.el7.x86_64 mysql-community-libs-compat-5.7.28-1.el7.x86_64 mysql-community-libs-5.7.28-1.el7.x86_64 mysql-community-server-5.7.28-1.el7.x86_64
```

- 卸载完后，用两个命令再次确认，mysql相关的包已经被卸载

  > 注意：确保mysql卸载干净，再继续往下操作

```shell
rpm -qa | grep -i mysql
yum list installed mysql*
```

![](/mysql\checkmysql.png)

### 4.4 删除mysql残留文件

- 查看mysql相关目录

```shell
find / -name mysql
```

> 根据自己的实际情况，删除find出来的目录

![](/mysql\findmysqldir.png)

```shell
rm -rf /var/lib/mysql/
rm -rf /usr/share/mysql/
rm -rf /etc/selinux/targeted/active/modules/100/mysql
```

- 另外删除文件：

```shell
rm -rf /root/.mysql_history
rm -f /var/log/mysqld.log
```

## 5。创建新用户并添加权限

### 5.1 使用root账号登录mysql

~~~shell
mysql -uroot -p123456
~~~

### 5.2 创建新用户

~~~shell
#低版本数据库
create user 'username'@'%' identified by '123456'
#高版本数据库
create user 'username'@'%' identified with mysql_native_password '123456'
#修改密码
alter user 'username'@'%' identified by '123456'
~~~

> '%' 所有情况都能访问
>
> 'localhost' 本机才能访问
>
> '192.168.1.110' 指定IP才能访问

### 5.3 给用户添加权限

~~~shell
#指定数据库
grant all privileges on dbname.* to 'username'@'%'
#全部数据库
grant all privileges on *.* to 'username'@'%'
~~~

> all 可以换成 select,delete,update,create,drop

### 5.4 删除用户

~~~shell
delete from mysql.user where user='username'
~~~

