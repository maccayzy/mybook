## Redis 环境安装

#### 一。下载安装包并解压

~~~shell
cd /yzy/soft
#在线安装上传下载工具
yum -y install lrzsz
wget http://download.redis.io/releases/redis-3.2.8.tar.gz
tar -zxvf redis-3.2.8.tar.gz -C /yzy/install
~~~

#### 二。安装C程序运行环境及最新TCL脚本语言

~~~shell
sudo yum -y install gcc-c++
sudo yum -y install tcl
~~~

#### 三。编译Redis

~~~shell
cd /yzy/install/redis-3.2.8
make test && make install
~~~

#### 四。修改Redis配置文件

~~~shell
cd /yzy/install/redis-3.2.8
mkdir -p /yzy/install/redis-3.2.8/logs
mkdir -p /yzy/install/redis-3.2.8/redisdata
vi redis.conf
~~~

~~~properties
bind node01
daemonize yes
pidfile /yzy/install/redis-3.2.8/redis_6397.pid
logfile "/yzy/install/redis-3.2.8/logs/redis.log"
dir /yzy/install/redis-3.2.8/redisdata
~~~

#### 五。启动Redis并连接客户端测试

~~~shell
cd /kkb/install/redis-3.2.8/src
redis-server ../redis.conf
redis-cli -h node01

node01:6379> set hello world
OK
node01:6379> get hello
"world"
node01:6379> 
~~~

#### 六。Redis主从复制架构

* 在另两台机器node02,node03重复上面步骤也安装好Redis

* 修改配置文件

  * node02 redis.conf

    ~~~properties
    bind node02
    daemonize yes
    pidfile /yzy/install/redis-3.2.8/redis_6397.pid
    logfile "/yzy/install/redis-3.2.8/logs/redis.log"
    dir /yzy/install/redis-3.2.8/redisdata
    slaveof node01 6379
    ~~~

  * node03 redis.conf

    ~~~properties
    bind node03
    daemonize yes
    pidfile /yzy/install/redis-3.2.8/redis_6397.pid
    logfile "/yzy/install/redis-3.2.8/logs/redis.log"
    dir /yzy/install/redis-3.2.8/redisdata
    slaveof node01 6379
    ~~~


* 分别启动node02跟node03的redis服务

  ~~~shell
  cd /yzy/install/redis-3.2.8/src
  redis-server ../redis.conf
  ~~~

  启动成功便可以实现Redis的主从复制了，node01可以读写操作，node02跟node03只支持读取操作

#### 七。Redis的Sentinel(哨兵)架构

​	Sentinel是Redis的高可用性解决方案，由一个或多个Sentinel实例组成的Sentinel系统可以监视任意多个主服务器，以及这些主服务器下的从服务器，当被监视的一个主服务进入下线状态时，自动将这个主服务器下的某个从服务器升级为新的主服务器

##### 1.  修改三台机器的Sentinel配置文件

~~~shell
cd /yzy/install/redis-3.2.8/
vi sentinel.conf
~~~

~~~properties
bind node01/node02/node03
daemonize yes
pidfile /yzy/install/redis-3.2.8/redis_sentinel.pid
logfile "/yzy/install/redis-3.2.8/logs/redis_sentinel.log"
dir /yzy/install/redis-3.2.8/redisdata
# mymaster 是自定义的服务器名
# node01代表监控的主服务器
# 6379 代表端口
# 2 代表只有二个或二个以上的哨兵认为主服务器不可用的时候,才会进行failover操作
sentinel mointor mymaster node01 6379 2
# mymaster 是自定义的服务器名
# 123456 服务器密码
sentinel auth-pass mymaster 123456
~~~
##### 2.  三台机器启动哨兵服务

~~~shell
cd /yzy/install/redis-3.2.8/src
redis-sentinel ../sentinel.conf
~~~

* 备注：最好是启动redis-server后,再启动redis-sentinel

#### 八。Redis集群
##### 1. 集群介绍
	 * Redis集群是一个提供在多个Redis节点之间共享数据的程序集
	
	 * 不支持同时处理多个键的Redis命令，因为这需要在多个节点间移动数据，会降低集群的性能

  * Redis集群通过分区来提供一定程度的可用性，即使一部分节点失效，集群也可以继续处理命令请求

  * 集群的优势：
    * 缓存永不宕机
    * 迅速恢复数据
    * Redis可以使用所有机器的内存，变相扩展性能
    * Redis计算能力通过增加服务器得到成倍提升，Redis的网络带宽也会随服务器和网卡的增加而成倍增长
    * Redis集群没有中心节点，所以没有性能瓶颈
    * 异步处理数据，实现快速读写

##### 2. 集群环境搭建

  * 最少要有三个主节点，每个主节点最少需要一个对应的从节点，所以最少要三主三从的配置

  * 先解压并编译

    ~~~
    cd /yzy/soft
    tar -zxvf redis-3.2.8.tar.gz -C	/yzy
    sudo yum -y install gcc-c++
    sudo yum -y install tcl
    cd /yzy/redis-3.2.8
    make test && make install
    ~~~

  * 创建不同实例的配置文件夹

    ~~~shell
    cd /yzy/redis-3.2.8
    mkdir -p /yzy/redis-3.2.8/clusters/7001
    mkdir -p /yzy/redis-3.2.8/clusters/7002
    mkdir -p /yzy/redis-3.2.8/clusters/7003
    mkdir -p /yzy/redis-3.2.8/clusters/7004
    mkdir -p /yzy/redis-3.2.8/clusters/7005
    mkdir -p /yzy/redis-3.2.8/clusters/7006
    
    mkdir -p /yzy/redis-3.2.8/logs
    mkdir -p /yzy/redis-3.2.8/redisdata/7001
    mkdir -p /yzy/redis-3.2.8/redisdata/7002
    mkdir -p /yzy/redis-3.2.8/redisdata/7003
    mkdir -p /yzy/redis-3.2.8/redisdata/7004
    mkdir -p /yzy/redis-3.2.8/redisdata/7005
    mkdir -p /yzy/redis-3.2.8/redisdata/7006
    
    cp /yzy/redis-3.2.8/redis.conf /yzy/redis-3.2.8/redisdata/7001
    cp /yzy/redis-3.2.8/redis.conf /yzy/redis-3.2.8/redisdata/7002
    cp /yzy/redis-3.2.8/redis.conf /yzy/redis-3.2.8/redisdata/7003
    cp /yzy/redis-3.2.8/redis.conf /yzy/redis-3.2.8/redisdata/7004
    cp /yzy/redis-3.2.8/redis.conf /yzy/redis-3.2.8/redisdata/7005
    cp /yzy/redis-3.2.8/redis.conf /yzy/redis-3.2.8/redisdata/7006
    ~~~

  * 修改六个配置文件

    * 7001配置文件

      ~~~properties
      bind node01
      port 7001
      cluster-enabled yes
      cluster-config-file nodes-7001.conf
      cluster-node-timeout 5000
      appendonly yes
      daemonize yes
      pidfile /yzy/redis-3.2.8/redis_7001.pid
      logfile "/yzy/redis-3.2.8/logs/7001.log"
      dir /yzy/redis-3.2.8/redisdata/7001
      ~~~

    * 7002配置文件

      ~~~properties
      bind node01
      port 7002
      cluster-enabled yes
      cluster-config-file nodes-7002.conf
      cluster-node-timeout 5000
      appendonly yes
      daemonize yes
      pidfile /yzy/redis-3.2.8/redis_7002.pid
      logfile "/yzy/redis-3.2.8/logs/7002.log"
      dir /yzy/redis-3.2.8/redisdata/7002
      ~~~

    * 7003配置文件

      ~~~properties
      bind node01
      port 7003
      cluster-enabled yes
      cluster-config-file nodes-7003.conf
      cluster-node-timeout 5000
      appendonly yes
      daemonize yes
      pidfile /yzy/redis-3.2.8/redis_7003.pid
      logfile "/yzy/redis-3.2.8/logs/7003.log"
      dir /yzy/redis-3.2.8/redisdata/7003
      ~~~
    * 7004配置文件

      ~~~properties
      bind node01
      port 7004
      cluster-enabled yes
      cluster-config-file nodes-7004.conf
      cluster-node-timeout 5000
      appendonly yes
      daemonize yes
      pidfile /yzy/redis-3.2.8/redis_7004.pid
      logfile "/yzy/redis-3.2.8/logs/7004.log"
      dir /yzy/redis-3.2.8/redisdata/7004
      ~~~

    * 7005配置文件

      ~~~properties
      bind node01
      port 7005
      cluster-enabled yes
      cluster-config-file nodes-7005.conf
      cluster-node-timeout 5000
      appendonly yes
      daemonize yes
      pidfile /yzy/redis-3.2.8/redis_7005.pid
      logfile "/yzy/redis-3.2.8/logs/7005.log"
      dir /yzy/redis-3.2.8/redisdata/7005
      ~~~

    * 7006配置文件

      ~~~properties
      bind node01
      port 7006
      cluster-enabled yes
      cluster-config-file nodes-7006.conf
      cluster-node-timeout 5000
      appendonly yes
      daemonize yes
      pidfile /yzy/redis-3.2.8/redis_7006.pid
      logfile "/yzy/redis-3.2.8/logs/7006.log"
      dir /yzy/redis-3.2.8/redisdata/7006
      ~~~

  * 启动redis进程

    ~~~shell
    /yzy/redis-3.2.8/src/redis-server /yzy/redis-3.2.8/clusters/7001/redis.conf
    /yzy/redis-3.2.8/src/redis-server /yzy/redis-3.2.8/clusters/7002/redis.conf
    /yzy/redis-3.2.8/src/redis-server /yzy/redis-3.2.8/clusters/7003/redis.conf
    /yzy/redis-3.2.8/src/redis-server /yzy/redis-3.2.8/clusters/7004/redis.conf
    /yzy/redis-3.2.8/src/redis-server /yzy/redis-3.2.8/clusters/7005/redis.conf
    /yzy/redis-3.2.8/src/redis-server /yzy/redis-3.2.8/clusters/7006/redis.conf
    ~~~

  * 安装ruby运行环境(redis集群启动需要)

    ~~~shell
    sudo yum -y install ruby
    sudo yum -y install rubygems
    gem -y install redis
    
    # 如果需要升级ruby
    cd /yzy/redis-3.2.8
    gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
    
    curl -sSL https://get.rvm.io | bash -s stable
    source /etc/profile.d/rvm.sh
    rvm list known
    rvm install 2.4.1
    ~~~

  * 创建redis集群

    ~~~shell
    cd /yzy/redis-3.2.8
    gem install redis
    src/redis-trib.rb create --replicas 1 node01:7001 node01:7002 node01:7003 node01:7004 node01:7005 node01:7006
    ~~~

    * 如果出错 （ERR Slot 741 is already busy）,就要清空所有节点的数据

      ~~~shell
      #每个节点都要进去清空一下
      src/redis-cli -h node01 -c -p 7001
      flushall
      cluster reset
      quit
      ~~~

##### 3. 集群管理

* 添加新节点作为主节点
    ~~~shell
    /yzy/redis-3.2.8/src/redis-server /yzy/redis-3.2.8/clusters/7007/redis.conf
    /yzy/redis-3.2.8/src/redis-trib.rb add-node node01:7007 node01:7001
    ~~~

* 添加新节点作为副节点    

  ~~~shell
  /yzy/redis-3.2.8/src/redis-server /yzy/redis-3.2.8/clusters/7008/redis.conf
  
  /yzy/redis-3.2.8/src/redis-trib.rb add-node --slave node01:7008 node01:7001
  ~~~

* 删除一个节点 (要先查到节点ID)

  ```shell
  #格式是：src/redis-trib del-node 127.0.0.1:7000 `<node-id>`
  
  /yzy/redis-3.2.8/src/redis-trib.rb del-node node01:7008 7c7b7f68bc56bf24cbb36b599d2e2d97b26c5540
  ```

* 重新分片

  ```shell
  #格式是：./redis-trib.rb reshard --from <node-id> --to <node-id> --slots <number of slots> --yes <host>:<port>
  
  ./redis-trib.rb reshard node01:7001
  ```




