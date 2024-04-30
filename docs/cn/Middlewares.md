🚧**装修中...**

# 目录

- [Apollo配置中心](#Apollo配置中心)



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
