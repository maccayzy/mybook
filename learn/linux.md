# Linux

#### 一. 常用Linux学习网址
> Linux命令用法查询： [Linux命令大全](http://man.linuxde.net/)
> 
> Linux全面知识：[鸟哥的Linux私房菜](http://cn.linux.vbird.org/)
> 
> Linux相关下载：[Linux下载站](http://www.linuxdown.net/)
> 
> Linux资讯论坛：[Linux中国开源社区](https://linux.cn/)
> 
> Linux在线学习网站：
>> 1. [实验楼](https://www.shiyanlou.com/) 需要注册，最大的优点是网站提供Linux在线开发环境,可以随便折腾
>> 2. [JSLinux ](http://bellard.org/jslinux/) 贼牛Ｂ的大神用JS写的Linux模拟器

#### 二. 常用Linux命令

1. **杀僵尸进程 终极绝招**

    > * ps -ef | grep defunct | grep -v grep | cut -b8-20 | xargs kill -9
2. **查看linux服务器cpu**

    > * cat /proc/cpuinfo |grep "processor"|wc -l
3. **查看数据库连接**

    > * netstat -n | grep 1521 | grep EST | wc -l
4. **查看端口**
    > 查看所有端口:
    >> * netstat -lnput
    > 
	>
    > 查看指定端口 :
    >> * netstat -lnput|grep 8888
    > 
    >
    >> a 表示所有 
    >> n 表示不查询dns 
    >> t 表示tcp协议 
    >> u 表示udp协议 
    >> p 表示查询占用的程序 
    >> l 表示查询正在监听的程序

5. **安装防火墙**
    > 检查是否已安装防火墙
    >> * systemctl status iptables
    >
    > 安装防火墙:
    >> *　yum install iptable-services
    >
    > 启动防火墙:
    >> *　systemctl start iptables.service
    >
    > 列出防火墙所有规则:
    >> * iptables -L -n

6. **防火墙规则设置**

    > * vim /etc/sysconfig/iptables
    >
    > 开放8888端口添加以下内容: 
    > * -A INPUT -m state --state NEW -p tcp -m tcp --dport 8888 -j ACCEPT
    >
    > 禁用8888端口添加以下内容:  
    > * -A INPUT -m state --state NEW -p tcp -m tcp --dport 8888 -j DROP
    >
    > 修改完保存退出:
    > * service iptables save 保存配置
    > * systemctl restart iptables.service 重启
    >
    > ***语法说明：***
    >
    > 　* -A: 追加到规则的最后一条
    > 　* -D：删除记录
    > 　* -I：添加到规则的第一条
    > 　* -p：（proto）规定通信协议，常见的协议有：tcp、udp、icmp、all
    > 　* -j：（jump）指定要跳转的目标，常见的目标有：ACCEPT（接收数据包）、DROP（丢弃数据包）、REJECT（重定向）三种，但是一般不适用重定向，会带来安全隐患 
