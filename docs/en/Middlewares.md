<p align="center">
  <a href="../en/Middlewares.md"><strong>English</strong></a> ｜ <a href="../cn/Middlewares.md">简体中文</a>
</p>

[← Contents](../../README.md)

# Contents

- [Apollo Configuration Center](#apollo-configuration-center)

The [MySQL](../cn/Middlewares.md#mysql) and [Redis](../cn/Middlewares.md#redis) sections are currently available in Chinese.



## Apollo Configuration Center

`Apollo` is a distributed configuration management center developed by Ctrip's framework R&D team. It manages application settings across environments and clusters, pushes changes to applications in real time, and provides access control and workflow management for microservices.

The server is built with `Spring Boot` and `Spring Cloud`. Once packaged, it can run without an extra application container such as `Tomcat`.

The .NET client does not depend on a framework and can run in any .NET runtime environment.

The following diagram shows the `Apollo` architecture:
![apollo](https://images.willxup.top/images/2024/03/29/m2vsj5.png)

`Apollo` has three services: `Portal`, `Config Service`, and `Admin Service`.

- `Portal`: Provides the `Apollo` management interface for developers.

- `Config Service`: Lets `Apollo` clients read settings and receive updates. It includes `Eureka`, `Config Service`, and `Meta Server` in one `JVM` process.

- `Admin Service`: Lets the `Apollo Portal` management interface modify and publish settings.

For multiple environments, deploy the `Portal` management interface once, and deploy `Config Service` and `Admin Service` once in each environment.


`GitHub`: [![GitHub Release](https://img.shields.io/github/v/release/apolloconfig/apollo)](https://github.com/apolloconfig/apollo)

`NuGet`: [![NuGet Version](https://img.shields.io/nuget/v/Com.Ctrip.Framework.Apollo.Configuration)](https://www.nuget.org/packages/Com.Ctrip.Framework.Apollo.Configuration)

Documentation: [https://www.apolloconfig.com/](https://www.apolloconfig.com/)

### Use the .NET Client

The `Apollo` client works with the standard .NET configuration API.

#### `NuGet` Dependency
```
Com.Ctrip.Framework.Apollo.Configuration
```

#### Configuration
Configure it in `Program.cs`.
```c#
builder.Host
	.ConfigureAppConfiguration((context, config) =>
	{
		// Configure Ctrip Apollo
		config.AddApollo(config.Build().GetSection("apollo"))
				  .AddDefault()
				  .AddNamespace("Test1") // Different namespace
				  .AddNamespace("Test2") // Different namespace
				  .AddNamespace("Test3"); // Different namespace
	});
```

#### Usage

Use `Microsoft.Extension` configuration directly.
```c#
public class TestService
{
	/// <summary>
	/// Configuration
	/// </summary>
	private readonly IConfiguration _configuration;

	public TestService(Microsoft.Extensions.Configuration.IConfiguration configuration)
	{
		_configuration = configuration;
	}

	public void Get()
	{
		// Config.ConnectString is a parameter configured in Apollo
		string connectStr = _configuration.GetValue<string>("Config.ConnectString");
	}
}
```

### Environments and Dependencies

> The following installation and deployment steps use `CentOS7`.



- `Apollo` supports isolated environments. The supported environments are:
	- `DEV`: Development
	- `FAT`: Test, equivalent to `alpha` (functional testing)
	- `UAT`: Integration, equivalent to `beta` (regression testing)
	- `PRO`: Production

- To install `Apollo`, you need:

	- Dependencies
		- Java Runtime `1.8+`
		- MySQL `5.6.5+`
	- Download `apollo`
		- [apollo-portal](https://github.com/apolloconfig/apollo/releases/download/v2.2.0/apollo-portal-2.2.0-github.zip)
		- [apollo-configservice](https://github.com/apolloconfig/apollo/releases/download/v2.2.0/apollo-configservice-2.2.0-github.zip)
		- [apollo-adminservice](https://github.com/apolloconfig/apollo/releases/download/v2.2.0/apollo-adminservice-2.2.0-github.zip)
	- Database files
		- [apolloportaldb.sql](https://github.com/apolloconfig/apollo/blob/master/scripts/sql/profiles/mysql-default/apolloportaldb.sql)
		- [apolloconfigdb.sql](https://github.com/apolloconfig/apollo/blob/master/scripts/sql/profiles/mysql-default/apolloconfigdb.sql)

### IP Addresses and Ports

To use two `Apollo` environments, first plan the IP addresses and ports for the services:
- `Portal`: `127.0.0.1:8070`
- `Dev Config Service`: `127.0.0.1:8071`
- `Dev Admin Service`: `127.0.0.1:8072`
- `Fat Config Service`: `127.0.0.1:8073`
- `Fat Admin Service`: `127.0.0.1:8074`

If the databases for the two environments and the server hosting `Apollo` are on different machines, configure them as needed in the same way.

### Database Configuration

#### ApolloPortalDB

- Create the `ApolloPortalDB` database.
- Run `apolloportaldb.sql`.
- Set `apollo.portal.envs` in the `ServerConfig` table to the list of supported environments (for example, `DEV,FAT`).

#### Development Environment: ApolloConfigDB

- Create the `ApolloConfigDB` database for the `dev` environment.
- Run `apolloconfigdb.sql`.
- Set `eureka.service.url` in the `ServerConfig` table to the `Config Service` address in the form `http://IP:Port` (for example, `http://127.0.0.1:8071/eureka/`).

#### Test Environment: ApolloConfigDB_Test

- Create the `ApolloConfigDB_Test` database for the `fat` environment.
- Run `apolloconfigdb.sql`.
- Set `eureka.service.url` in the `ServerConfig` table to the `Config Service` address in the form `http://IP:Port` (for example, `http://127.0.0.1:8073/eureka/`).

### Configure Portal

Deploy `Portal`, the management interface, once.
The default `Portal` account is `apollo` and its password is `admin`.

#### Directory Structure

Extract `apollo-portal-2.2.0-github.zip` to get this directory structure:
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

#### File Configuration

- `apollo-portal.conf`
> Configure the `portal` log file location.
```bash
## PID directory
PID_FOLDER=.
## Log directory
LOG_FOLDER=/data/apollo/logs/portal/
## Log name
LOG_FILENAME=apollo-portal.console.log
```

- `config/apollo-env.properties`
> Configure the environments supported by `Portal`.
```bash
#local.meta=http://localhost:8080
dev.meta=http://127.0.0.1:8071 ## Configure the dev environment (Dev Config Service IP and port)
fat.meta=http://127.0.0.1:8073 ## Configure the fat environment (Fat Config Service IP and port)
#uat.meta=http://fill-in-uat-meta-server:8080
#pro.meta=http://fill-in-pro-meta-server:8080
```

- `config/application-github.properties`
> Configure the database used by `Portal`.
```bash
spring.datasource.url = jdbc:mysql://127.0.0.1:3306/ApolloPortalDB?characterEncoding=utf8
spring.datasource.username = your_username
spring.datasource.password = your_password
```

- `scripts/startup.sh`
> Configure the `Portal` startup settings.
```bash
## Configure the log location
LOG_DIR=/data/apollo/logs/portal
## Configure the Portal port
SERVER_PORT=${SERVER_PORT:=8070}
## Configure Portal JAVA startup options; these are usually left unchanged
export JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=4 -XX:MaxTenuringThreshold=9 -XX:+DisableExplicitGC -XX:+ScavengeBeforeFullGC -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+ExplicitGCInvokesConcurrent -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Duser.timezone=Asia/Shanghai -Dclient.encoding.override=UTF-8 -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom"
```

 - Permissions
 > Make the `sh` scripts in the `scripts` directory executable.
```bash
chmod +x *.sh
```

### Configure Config Service

`Config Service` provides configuration reads and updates, so deploy it in each environment.

#### Directory Structure

Extract `apollo-configservice-2.2.0-github` to get the following structure. You can rename the directory for different environments.
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

#### File Configuration

- `apollo-configservice.conf`
> Configure the `Config Service` log file location.
```bash
## PID directory
PID_FOLDER=.
## Log directory
LOG_FOLDER=/data/apollo/logs/dev
## Log name
LOG_FILENAME=dev-apollo-configservice.console.log
```

- `config/application-github.properties`
> Configure the database used by `Config Service`.
```bash
spring.datasource.url = jdbc:mysql://127.0.0.1:3306/ApolloConfigDB?characterEncoding=utf8
spring.datasource.username = your_username
spring.datasource.password = your_password
```

- `scripts/startup.sh`
> Configure the `Config Service` startup settings.
```bash
## Configure the log location
LOG_DIR=/data/apollo/logs/dev
## Configure the Portal port
SERVER_PORT=${SERVER_PORT:=8071}
## Configure Portal JAVA startup options; these are usually left unchanged
export JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=4 -XX:MaxTenuringThreshold=9 -XX:+DisableExplicitGC -XX:+ScavengeBeforeFullGC -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+ExplicitGCInvokesConcurrent -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Duser.timezone=Asia/Shanghai -Dclient.encoding.override=UTF-8 -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom"
```

 - Permissions
 > Make the `sh` scripts in the `scripts` directory executable.
```bash
chmod +x *.sh
```

#### Local Deployment Issue

`Apollo` clients and `Portal` get the service address (`IP+Port`) from `Config Service` and connect to it directly. The `eureka` service discovery page may show a public IP instead, which can prevent clients from connecting to `Config Service`. Update `JAVA_OPTS` in `startup.sh`:
```shell
## Find JAVA_OPTS and append -Deureka.instance.ip-address=127.0.0.1
JAVA_OPTS="$JAVA_OPTS -Deureka.instance.ip-address=127.0.0.1"
```


### Configure Admin Service

#### Directory Structure

Extract `apollo-adminservice-2.2.0-github` to get the following structure. You can rename the directory for different environments.
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

#### File Configuration

- `apollo-adminservice.conf`
> Configure the `Admin Service` log file location.
```bash
## PID directory
PID_FOLDER=.
## Log directory
LOG_FOLDER=/data/apollo/logs/dev
## Log name
LOG_FILENAME=dev-apollo-configservice.console.log
```

- `config/application-github.properties`
> Configure the database used by `Admin Service`.
```bash
spring.datasource.url = jdbc:mysql://127.0.0.1:3306/ApolloConfigDB?characterEncoding=utf8
spring.datasource.username = your_username
spring.datasource.password = your_password
```

- `scripts/startup.sh`
> Configure the `Admin Service` startup settings.
```bash
## Configure the log location
LOG_DIR=/data/apollo/logs/dev
## Configure the Portal port
SERVER_PORT=${SERVER_PORT:=8072}
## Configure Portal JAVA startup options; these are usually left unchanged
export JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=4 -XX:MaxTenuringThreshold=9 -XX:+DisableExplicitGC -XX:+ScavengeBeforeFullGC -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+ExplicitGCInvokesConcurrent -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Duser.timezone=Asia/Shanghai -Dclient.encoding.override=UTF-8 -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom"
```

 - Permissions
 > Make the `sh` scripts in the `scripts` directory executable.
```bash
chmod +x *.sh
```

#### Local Deployment Issue

`Apollo` clients and `Portal` get the service address (`IP+Port`) from `Config Service` and connect to it directly. The `eureka` service discovery page may show a public IP instead, which can prevent clients from connecting to `Config Service`. Update `JAVA_OPTS` in `startup.sh`:
```shell
## Find JAVA_OPTS and append -Deureka.instance.ip-address=127.0.0.1
JAVA_OPTS="$JAVA_OPTS -Deureka.instance.ip-address=127.0.0.1"
```

### Deployment

All three `Apollo` services need to start, so startup scripts can help.

#### Startup Script
```bash
#!/bin/sh

#description: start apollo

# Development environment
cd /data/apollo/dev-configservice/scripts
./startup.sh

cd /data/apollo/dev-adminservice/scripts
./startup.sh


# Test environment
cd /data/apollo/fat-configservice/scripts
./startup.sh

cd /data/apollo/fat-adminservice/scripts
./startup.sh


#portal
cd /data/apollo/portal/scripts
./startup.sh
```

#### Stop Script
```bash
#!/bin/sh

#description: stop apollo

# Development environment
cd /data/apollo/dev-configservice/scripts
./shutdown.sh

cd /data/apollo/dev-adminservice/scripts
./shutdown.sh


# Test environment
cd /data/apollo/fat-configservice/scripts
./shutdown.sh

cd /data/apollo/fat-adminservice/scripts
./shutdown.sh


#portal
cd /data/apollo/portal/scripts
./shutdown.sh
```

#### Systemd Startup Script

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
