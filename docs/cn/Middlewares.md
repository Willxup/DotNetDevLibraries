# 目录

- [Apollo配置中心](#Apollo配置中心)
- [MySQL](#MySQL)
- [Redis](#Redis)



## Apollo配置中心

`Apollo`（阿波罗）是一款可靠的分布式配置管理中心，诞生于携程框架研发部，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

服务端基于`Spring Boot`和`Spring Cloud`开发，打包后可以直接运行，不需要额外安装`Tomcat`等应用容器。

`.Net`客户端不依赖任何框架，能够运行于所有`.Net`运行时环境。

`Apollo`整体架构如下图：
![apollo](https://images.willxup.top/images/2024/03/29/m2vsj5.png)

`Apollo`配置中心被分为`portal`，`Config Service`，`Admin Service`三个服务

- `Portal`： 提供`Apollo`配置中心的管理界面，服务对象是开发人员

- `Config Service`：提供配置的读取、推送等功能，服务对象是`Apollo`客户端，该服务包含`Eureka`，`Config Service`和`Meta Server`，他们在同一个`JVM`进程中。

- `Admin Service`：提供配置的修改、发布等功能，服务对象是`Apollo Portal`管理界面

如果需要`Apollo`配置中心支持多个环境，`Portal`管理界面只需要部署一处，`Config Service`和`Admin Service`在每一个环境部署一次。


`Github`：[![GitHub Release](https://img.shields.io/github/v/release/apolloconfig/apollo)](https://github.com/apolloconfig/apollo)

`Nuget`：[![NuGet Version](https://img.shields.io/nuget/v/Com.Ctrip.Framework.Apollo.Configuration)](https://www.nuget.org/packages/Com.Ctrip.Framework.Apollo.Configuration)

文档地址：[https://www.apolloconfig.com/](https://www.apolloconfig.com/)

### .NET客户端使用

`Apollo`的客户端使用起来非常简单，直接使用`.NET`原生获取配置的方式即可

#### `Nuget`依赖
```
Com.Ctrip.Framework.Apollo.Configuration
```

#### 配置
在`Program.cs`文件中进行配置
```c#
builder.Host
	.ConfigureAppConfiguration((context, config) =>
	{
		//设置携程Apollo配置中心
		config.AddApollo(config.Build().GetSection("apollo"))
				  .AddDefault()
				  .AddNamespace("Test1") //不同的namespace
				  .AddNamespace("Test2") //不同的namespace
				  .AddNamespace("Test3"); //不同的namespace
	});
```

#### 使用

直接使用`Microsoft.Extension`的配置即可。
```c#
public class TestService
{
	/// <summary>
	/// 配置
	/// </summary>
	private readonly IConfiguration _configuration;
	
	public TestService(Microsoft.Extensions.Configuration.IConfiguration configuration)
	{
		_configuration = configuration;
	}

	public void Get()
	{
		//Config.ConnectString是在apollo中配置的参数
		string connectStr = _configuration.GetValue<string>("Config.ConnectString");
	}
}
```

### 环境与依赖

> 以下内容是基于`CentOS7`系统进行安装部署。



- `Apollo`配置中心支持不同的环境，每个环境互相隔离。`Apollo`目前支持以下环境：
	- `DEV` 开发环境
	- `FAT` 测试环境，相当于`alpha`环境(功能测试)
	- `UAT` 集成环境，相当于`beta`环境（回归测试）
	- `PRO` 生产环境

- 要安装`Apollo`配置中心需要以下：

	- 依赖 
		- [[Java Runtime]] `1.8+`
		- [[MySQL]] `5.6.5+`
	- 下载`apollo`
		- [apollo-portal](https://github.com/apolloconfig/apollo/releases/download/v2.2.0/apollo-portal-2.2.0-github.zip)
		- [apollo-configservice](https://github.com/apolloconfig/apollo/releases/download/v2.2.0/apollo-configservice-2.2.0-github.zip)
		- [apollo-adminservice](https://github.com/apolloconfig/apollo/releases/download/v2.2.0/apollo-adminservice-2.2.0-github.zip)
	- 数据库文件
		- [apolloportaldb.sql](https://github.com/apolloconfig/apollo/blob/master/scripts/sql/profiles/mysql-default/apolloportaldb.sql)
		- [apolloconfigdb.sql](https://github.com/apolloconfig/apollo/blob/master/scripts/sql/profiles/mysql-default/apolloconfigdb.sql)

### IP端口配置

让`Apollo`配置中心支持两套环境，需要先规划配置Apollo服务的IP和端口：
- `Portal`：`127.0.0.1：8070`
- `Dev Config Service`：`127.0.0.1：8071`
- `Dev Admin Service`：`127.0.0.1：8072`
- `Fat Config Service`：`127.0.0.1：8073`
- `Fat Admin Service`：`127.0.0.1：8074`

如果两个环境的数据库和准备部署`Apollo`的服务器不是同一个，可按需配置，配置方式也是相同的。

### 数据库配置

#### ApolloPortalDB

- 创建`ApolloPortalDB`数据库
- 运行`apolloportaldb.sql`
- 配置`ServerConfig`表中`apollo.portal.envs`的值，该值表示可支持的环境列表(例：`DEV,FAT`)

#### 开发环境ApolloConfigDB

- 创建`ApolloConfigDB`数据库`dev`环境
- 运行`apolloconfigdb.sql`
- 配置`ServerConfig`表中`eureka.service.url`的值，该值就是`Config Service`的`http://IP:Port`地址(例：`http://127.0.0.1:8071/eureka/`)

#### 测试环境ApolloConfigDB_Test

- 创建`ApolloConfigDB_Test`数据库作为`fat`环境
- 运行`apolloconfigdb.sql`
- 配置`ServerConfig`表中`eureka.service.url`的值，该值就是`Config Service`的`http://IP:Port`地址(例：`http://127.0.0.1:8073/eureka/`)

### 配置Portal

`Portal`作为管理界面，只需要部署一次即可。
`Portal`首次登陆的默认账号为`apollo`，密码为`admin`。

#### 目录结构

解压`apollo-portal-2.2.0-github.zip`文件，将会得到以下结构目录：
```bash
- apollo-portal-2.2.0-github
	- config
		- apollo-env.properties
		- application.properties
		- application-github.properties
	- scripts
		- shutdown.sh
		- startup.sh
	- apollo-portal.conf
	- apollo-portal.jar
	- apollo-portal-2.2.0.jar
	- apollo-portal-2.2.0-sources.jar
```

#### 文件配置

- `apollo-portal.conf`
> 配置`portal`的日志文件存储位置
```bash
## pid目录
PID_FOLDER=.
## 日志目录
LOG_FOLDER=/data/apollo/logs/portal/
## 日志名称
LOG_FILENAME=apollo-portal.console.log
```

- `config/apollo-env.properties`
> 配置`Portal`支持的环境
```bash
#local.meta=http://localhost:8080
dev.meta=http://127.0.0.1:8071 ## 配置dev环境(Dev Config Service的IP端口)
fat.meta=http://127.0.0.1:8073 ## 配置fat环境(Fat Config Service的IP端口)
#uat.meta=http://fill-in-uat-meta-server:8080
#pro.meta=http://fill-in-pro-meta-server:8080
```

- `config/application-github.properties`
> 配置`Portal`使用的数据库信息
```bash
spring.datasource.url = jdbc:mysql://127.0.0.1:3306/ApolloPortalDB?characterEncoding=utf8
spring.datasource.username = your_username
spring.datasource.password = your_password
```

- `scripts/startup.sh`
> 配置`Portal`的启动设置
```bash
## 配置日志存放位置
LOG_DIR=/data/apollo/logs/portal
## 配置Portal端口
SERVER_PORT=${SERVER_PORT:=8070}
## 配置Portal启动时JAVA选项，一般不进行改动
export JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=4 -XX:MaxTenuringThreshold=9 -XX:+DisableExplicitGC -XX:+ScavengeBeforeFullGC -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+ExplicitGCInvokesConcurrent -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Duser.timezone=Asia/Shanghai -Dclient.encoding.override=UTF-8 -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom"
```

 - 授权
 > 赋予`scripts`目录下的`sh`脚本可执行权限
```bash
chmod +x *.sh
```

### 配置Config Service

`Config Service`提供配置的读取、推送等功能，所以再不同的环境下，需要再次部署该服务。

#### 目录结构

解压`apollo-configservice-2.2.0-github`文件，将会得到以下结构目录，可修改目录名称以支持不同的环境
```bash
- apollo-configservice-2.2.0-github(dev-configservice)
	- config
		- application.properties
		- application-github.properties
	- scripts
		- shutdown.sh
		- startup.sh
	- apollo-configservice.conf
	- apollo-configservice-2.2.0.jar
	- apollo-configservice-2.2.0-sources.jar
```

#### 文件配置

- `apollo-configservice.conf`
> 配置`Config Service`的日志文件存储位置
```bash
## pid目录
PID_FOLDER=.
## 日志目录
LOG_FOLDER=/data/apollo/logs/dev
## 日志名称
LOG_FILENAME=dev-apollo-configservice.console.log
```

- `config/application-github.properties`
> 配置`Config Service`使用的数据库信息
```bash
spring.datasource.url = jdbc:mysql://127.0.0.1:3306/ApolloConfigDB?characterEncoding=utf8
spring.datasource.username = your_username
spring.datasource.password = your_password
```

- `scripts/startup.sh`
> 配置`Config Service`的启动设置
```bash
## 配置日志存放位置
LOG_DIR=/data/apollo/logs/dev
## 配置Portal端口
SERVER_PORT=${SERVER_PORT:=8071}
## 配置Portal启动时JAVA选项，一般不进行改动
export JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=4 -XX:MaxTenuringThreshold=9 -XX:+DisableExplicitGC -XX:+ScavengeBeforeFullGC -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+ExplicitGCInvokesConcurrent -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Duser.timezone=Asia/Shanghai -Dclient.encoding.override=UTF-8 -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom"
```

 - 授权
 > 赋予`scripts`目录下的`sh`脚本可执行权限
```bash
chmod +x *.sh
```

#### 本地部署出现的问题

`Apollo`客户端和`Portal`会从`Config Service`获取服务的地址（`IP+Port`），然后通过**服务地址**直接访问。而从`eureka`服务发现的页面会发现，服务地址变成了`公网IP`，这可能会导致`Apollo`客户端无法连接上`Config Service`，所以需要修改`startup.sh`配置`JAVA_OPTS`。
```shell
## 可以直接找到JAVA_OPTS，在最末尾加上 -Deureka.instance.ip-address=127.0.0.1
JAVA_OPTS="$JAVA_OPTS -Deureka.instance.ip-address=127.0.0.1"
```


### 配置Admin Service

#### 目录结构

解压`apollo-adminservice-2.2.0-github`文件，将会得到以下结构目录，可修改目录名称以支持不同的环境
```bash
- apollo-adminservice-2.2.0-github(dev-adminservice)
	- config
		- application.properties
		- application-github.properties
	- scripts
		- shutdown.sh
		- startup.sh
	- apollo-adminservice.conf
	- apollo-adminservice-2.2.0.jar
	- apollo-adminservice-2.2.0-sources.jar
```

#### 文件配置

- `apollo-adminservice.conf`
> 配置`Admin Service`的日志文件存储位置
```bash
## pid目录
PID_FOLDER=.
## 日志目录
LOG_FOLDER=/data/apollo/logs/dev
## 日志名称
LOG_FILENAME=dev-apollo-configservice.console.log
```

- `config/application-github.properties`
> 配置`Admin Service`使用的数据库信息
```bash
spring.datasource.url = jdbc:mysql://127.0.0.1:3306/ApolloConfigDB?characterEncoding=utf8
spring.datasource.username = your_username
spring.datasource.password = your_password
```

- `scripts/startup.sh`
> 配置`Admin Service`的启动设置
```bash
## 配置日志存放位置
LOG_DIR=/data/apollo/logs/dev
## 配置Portal端口
SERVER_PORT=${SERVER_PORT:=8072}
## 配置Portal启动时JAVA选项，一般不进行改动
export JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=4 -XX:MaxTenuringThreshold=9 -XX:+DisableExplicitGC -XX:+ScavengeBeforeFullGC -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+ExplicitGCInvokesConcurrent -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Duser.timezone=Asia/Shanghai -Dclient.encoding.override=UTF-8 -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom"
```

 - 授权
 > 赋予`scripts`目录下的`sh`脚本可执行权限
```bash
chmod +x *.sh
```

#### 本地部署出现的问题

`Apollo`客户端和`Portal`会从`Config Service`获取服务的地址（`IP+Port`），然后通过**服务地址**直接访问。而从`eureka`服务发现的页面会发现，服务地址变成了`公网IP`，这可能会导致`Apollo`客户端无法连接上`Config Service`，所以需要修改`startup.sh`配置`JAVA_OPTS`。
```shell
## 可以直接找到JAVA_OPTS，在最末尾加上 -Deureka.instance.ip-address=127.0.0.1
JAVA_OPTS="$JAVA_OPTS -Deureka.instance.ip-address=127.0.0.1"
```

### 部署

`Apollo`三个服务都需要启动，所以编写脚本启动是比较好的方式。

#### 启动脚本
```bash
#!/bin/sh

#description: start apollo

#开发环境
cd /data/apollo/dev-configservice/scripts
./startup.sh

cd /data/apollo/dev-adminservice/scripts
./startup.sh


#测试环境
cd /data/apollo/fat-configservice/scripts
./startup.sh

cd /data/apollo/fat-adminservice/scripts
./startup.sh


#portal
cd /data/apollo/portal/scripts
./startup.sh
```

#### 停止脚本
```bash
#!/bin/sh

#description: stop apollo

#开发环境
cd /data/apollo/dev-configservice/scripts
./shutdown.sh

cd /data/apollo/dev-adminservice/scripts
./shutdown.sh


#测试环境
cd /data/apollo/fat-configservice/scripts
./shutdown.sh

cd /data/apollo/fat-adminservice/scripts
./shutdown.sh


#portal
cd /data/apollo/portal/scripts
./shutdown.sh
```

#### Systemd自启脚本

```bash
[Unit]
Description=auto run Apollo
After=network.target

[Service]
Type=simple
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/java/jdk1.8.0_281/bin"
User=root
ExecStart=/data/apollo/start.sh
TimeoutStartSec=0
[Install]
WantedBy=multi-user.target
```



# MySQL

> 以下内容是基于`CentOS7`系统进行安装部署。



## 安装MySQL

在官网下载[MySQL安装包](https://dev.mysql.com/downloads/mysql/)，下载 `mysql-8.0.21-1.el7.x86_64.rpm-bundle.tar`

```shell
#解压压缩包
tar -xvf  mysql-8.0.20-1.el7.x86_64.rpm-bundle.tar 

#安装Common
rpm -ivh mysql-community-common-8.0.21-1.el7.x86_64.rpm --nodeps --force

#安装lib
rpm -ivh mysql-community-libs-8.0.21-1.el7.x86_64.rpm

#如果遇到error: Failed dependencies: mariadb-libs is obsoleted by mysql-community-libs-8.0.21-1.el7.x86_64，清除之前安装过的依赖mysql-libs
yum remove mysql-libs

#安装client
rpm -ivh mysql-community-client-8.0.21-1.el7.x86_64.rpm

#安装server
rpm -ivh mysql-community-server-8.0.21-1.el7.x86_64.rpm

#如果遇到error: Failed dependencies:libaio.so.1()(64bit) is needed by mysql-community-server... 需要安装libaio.so依赖
yum install -y libaio

```



## 启动MySQL

```shell
#初始化数据库
mysqld --initialize
#设置文件所有者和文件关联组，拥有者皆设为mysql，群体的使用者mysql
chown mysql:mysql -R /var/lib/mysql 
#开启服务
systemctl start mysqld.service
#服务状态
systemctl status  mysqld.service
#服务自启
systemctl enable mysqld.service
```



## 配置mysql

> `mysql`的配置文件路径：`/etc/my.cnf`

```shell
#For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysql] #设置客户端
# 设置mysql客户端默认字符集
#default-character-set=utf8mb4

[mysqld] #设置服务端
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove the leading "# " to disable binary logging
# Binary logging captures changes between backups and is enabled by
# default. It's default setting is log_bin=binlog
# disable_log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
#
# Remove leading # to revert to previous value for default_authentication_plugin,
# this will increase compatibility with older clients. For background, see:
# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
# default-authentication-plugin=mysql_native_password


#设置端口号，默认3306
#port=3306

# 设置mysql的安装目录
#basedir=/usr/local/mysql

# 设置mysql数据库的数据的存放目录
datadir=/usr/local/mysql/data
socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# 允许最大连接数
# max_connections=500

# 服务端使用的字符集默认为8比特编码的latin1字符集
#character-set-server=utf8mb4

# 创建新表时将使用的默认存储引擎
#default-storage-engine=INNODB
#lower_case_table_names=1
#max_allowed_packet=16M


#跳过密码验证，之后会有获取密码的方式，如果找不到密码可以先设置跳过密码验证，改过密码后再注释掉
#skip-grant-tables

#设置performance_schema_max_table_instances 默认情况下为12500(修改后减少的内存占用很少)
#performance_schema_max_table_instances=150

#设置table_definition_cache默认情况下为2000(修改后减少内存明显)
#table_definition_cache=400

#设置table_open_cache默认情况下为4000(修改后减少的内存占用很少)
#table_open_cache=2000

#设置innodb_buffer_pool_size默认情况下为128M(修改后减少内存不明显,减少5M左右)
#innodb_buffer_pool_size=64M
```



## 数据库设置

```shell
#获取初始密码
grep password /var/log/mysqld.log 

#登录
mysql -uroot -p 

#设置密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456'; 

#重新加载权限表
FLUSH PRIVILEGES;

#开启远程登录授权
CREATE USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456'; 
#赋权命令 以下命令二选一 
# all privileges 当前用户所有权限 
# *.*赋予所有权限 
# with grant option允许级联赋权
#1.任何主机访问都可以连接 '%'
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
#2.指定主机访问可连接
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.0.1' WITH GRANT OPTION;

#重新加载权限表
FLUSH PRIVILEGES;
```



# Redis



## 安装Redis

> 以下内容是基于`CentOS7`系统进行安装部署。

### yum安装

```shell

#下载fedora的epel仓库
yum install epel-release
yum install -y gcc-c++

#搜索redis 两种方式都可行
yum search redis --showduplicates
yum list redis --showduplicates

#安装redis，可以选择yum搜索redis的结果进行安装
yum install redis

#设置开机自启  
systemctl enable redis

#开启服务
systemctl start redis
```



### 编译安装

####  安装redis
> `Redis`官网：[https://download.redis.io/releases](https://download.redis.io/releases)
>
> `Redis`安装包：[https://download.redis.io/releases/redis-6.2.6.tar.gz](https://download.redis.io/releases/redis-6.2.6.tar.gz)

```shell
#1.下载redis
wget https://download.redis.io/releases/redis-6.2.6.tar.gz

#2.解压
tar -zxvf redis-6.2.6.tar.gz

#3.进入解压后的文件编译安装 [[make编译]]
make

#如果遇到 zmalloc.h:50:31: 致命错误：jemalloc/jemalloc.h：没有那个文件或目录 需要指定MALLOC=libc
make MALLOC=libc

#4.安装  指定安装目录 PREFIX=/usr/local/redis
make install 
```

#### 测试redis

```bash
#6.启动redis并指定redis.conf文件
./redis-server /etc/redis/redis.conf

#进入redis，测试是否可用
redis-cli
#验证(用户名 密码)
auth password

#杀死redis进程
kill -9 xxx
```

#### 配置service文件

可参考[[服务管理]]
```bash
#7.创建redis的unit服务文件
cd /etc/systemd/system
touch redis.service
##########################################
[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf 
#--supervised systemd
#ExecStop=/usr/libexec/redis-shutdown
Type=forking
#User=redis
#Group=redis
#RuntimeDirectory=redis

[Install]
WantedBy=multi-user.target
##########################################

#重启服务
systemctl restart redis 
```



## 配置redis

```shell
#如果没有redis配置目录，就创建一个
mkdir /etc/redis

#编辑redis配置文件
#将redis.conf移至/etc/redis路径下
vim /etc/redis/redis.conf 
```

根据下面的参数解释，配置`redis`
```shell
#可查看redis.conf中文版，阿里云盘保存
bind 127.0.0.1 #注释该配置。配置可访问的IP，当保护模式打开时，如果未设置密码，需要配置外部ip使外部ip可访问
port 6379 #redis端口号
daemonize yes  #开启守护进程模式，redis后台运行(如果docker部署的话,要设置为no,否则后台运行会冲突)
protected-mode yes #保护模式开启。保护模式开启时在【没有密码】时通过bind ip控制外网可访问的ip
dir /opt/redis #保存redis数据的目录
logfile /var/log/redis/redis-server.log #保存redis日志的位置 
requirepass 123456  #设置密码
appendonly yes #持久化存储
databases 16 #redis数据库数量 默认16个数据库，可按需设置
```
