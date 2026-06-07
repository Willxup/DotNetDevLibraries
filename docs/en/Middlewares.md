<p align="center">
  <a href="../en/Middlewares.md"><strong>English</strong></a> ｜ <a href="../cn/Middlewares.md">简体中文</a>
</p>

[← Contents](../../README.md)

# Contents

- [Apollo Configuration Center](#apollo-configuration-center)
- [MySQL](#mysql)

The [Redis](../cn/Middlewares.md#redis) section is currently available in Chinese.



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

# MySQL

> The following installation and deployment steps use `CentOS7`.



## Install MySQL

Download the [MySQL installation package](https://dev.mysql.com/downloads/mysql/) from the official site: `mysql-8.0.21-1.el7.x86_64.rpm-bundle.tar`.

```shell
# Extract the archive
tar -xvf  mysql-8.0.20-1.el7.x86_64.rpm-bundle.tar

# Install Common
rpm -ivh mysql-community-common-8.0.21-1.el7.x86_64.rpm --nodeps --force

# Install lib
rpm -ivh mysql-community-libs-8.0.21-1.el7.x86_64.rpm

# If you see error: Failed dependencies: mariadb-libs is obsoleted by mysql-community-libs-8.0.21-1.el7.x86_64, remove the previously installed mysql-libs dependency
yum remove mysql-libs

# Install client
rpm -ivh mysql-community-client-8.0.21-1.el7.x86_64.rpm

# Install server
rpm -ivh mysql-community-server-8.0.21-1.el7.x86_64.rpm

# If you see error: Failed dependencies:libaio.so.1()(64bit) is needed by mysql-community-server..., install the libaio.so dependency
yum install -y libaio

```



## Start MySQL

```shell
# Initialize the database
mysqld --initialize
# Set the file owner and group to mysql
chown mysql:mysql -R /var/lib/mysql
# Start the service
systemctl start mysqld.service
# Check service status
systemctl status  mysqld.service
# Enable the service at startup
systemctl enable mysqld.service
```



## Configure MySQL

> The `mysql` configuration file is at `/etc/my.cnf`.

```shell
#For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysql] # Configure the client
# Set the default MySQL client character set
#default-character-set=utf8mb4

[mysqld] # Configure the server
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


# Set the port (default: 3306)
#port=3306

# Set the MySQL installation directory
#basedir=/usr/local/mysql

# Set the MySQL data directory
datadir=/usr/local/mysql/data
socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# Maximum allowed connections
# max_connections=500

# The server character set defaults to 8-bit latin1
#character-set-server=utf8mb4

# Default storage engine for new tables
#default-storage-engine=INNODB
#lower_case_table_names=1
#max_allowed_packet=16M


# The password can be retrieved later. If it cannot be found, skip verification temporarily; comment this out after changing it
#skip-grant-tables

# Set performance_schema_max_table_instances (default: 12500; changing it saves little memory)
#performance_schema_max_table_instances=150

# Set table_definition_cache (default: 2000; changing it reduces memory use noticeably)
#table_definition_cache=400

# Set table_open_cache (default: 4000; changing it saves little memory)
#table_open_cache=2000

# Set innodb_buffer_pool_size (default: 128M; changing it saves little memory, about 5M)
#innodb_buffer_pool_size=64M
```



## Database Settings

```shell
# Get the initial password
grep password /var/log/mysqld.log

# Log in
mysql -uroot -p

# Set the password
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';

# Reload the privilege tables
FLUSH PRIVILEGES;

# Enable remote login
CREATE USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
# Grant privileges: choose one of the following commands
# all privileges: all privileges for the current user
# *.*: grant all privileges
# with grant option: allow privileges to be granted to others
# 1. Allow connections from any host '%'
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
# 2. Allow connections from a specified host
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.0.1' WITH GRANT OPTION;

# Reload the privilege tables
FLUSH PRIVILEGES;
```
