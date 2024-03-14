
🚧**装修中...**

# CAP

## 介绍

CAP 是一个EventBus，同时也是一个在微服务或者SOA系统中解决分布式事务问题的一个框架。它有助于创建可扩展，可靠并且易于更改的微服务系统。CAP除了可以作为消息队列使用外，还可以解决分布式事务的同步问题。

- CAP支持的数据库：`MySQL`, `SQLServer`, `PostgreSql`, `MongoDB`

- CAP支持的传输：`Kafka`, `RabbitMQ`, `Redis` 等

`Github`：[![GitHub Release](https://img.shields.io/github/v/release/dotnetcore/CAP)](https://github.com/dotnetcore/CAP)

`Nuget`：[![NuGet Version](https://img.shields.io/nuget/v/DotNetCore.CAP)](https://www.nuget.org/packages/DotNetCore.CAP)

文档：[https://cap.dotnetcore.xyz/](https://cap.dotnetcore.xyz/)

## 快速上手

### Nuget包

```c#
DotNetCore.CAP
DotNetCore.CAP.MySql //以Mysql为例
DotNetCore.CAP.RedisStreams //以redis为例

DotNetCore.CAP.Dashboard //面板
```


### IOC注入

```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

builder.Services.AddCap(options => 
{
	//使用MySQL作为存储
	options.UseMySql(config => 
	{
		//从appsettings中获取MySQL连接字符串
		config.ConnectionString = configuration.GetValue<string>("Database.ConnectionString");
		
		//CAP生成的表名前缀
		config.TableNamePrefix = "SYS_CAP";
	});

	//使用Redis Streams 作为消息传输器
	options.UseRedis(config =>
	{
		//从appsettings中获取Redis连接字符串
		ConfigurationOptions opt = ConfigurationOptions.Parse(configuration.GetValue<string>("Redis.ConnectionString"));
		//默认数据库
		opt.DefaultDatabase = configuration.GetValue<int>("Redis.DbNum");
		
		config.Configuration = opt;
	});


	//设置失败重试次数
	options.FailedRetryCount = 5;

	//执行失败以后,下一次执行的时间间隔(秒)
	options.FailedRetryInterval = 60;

	//设置处理成功的数据在数据库中保存的时间（秒），为保证系统性能，数据会定期清理。
	options.SucceedMessageExpiredAfter = 604800; //7天

	//设置处理失败的数据再数据库中保存的时间（秒）
	options.FailedMessageExpiredAfter = 604800; //7天

	//分组前缀
	options.GroupNamePrefix = "FastAdminAPI";
	//默认分组名称
	options.DefaultGroupName = "DefultGroup";

	////Topic前缀
	//x.TopicNamePrefix = "Topic.";

	//失败超过重试次数后的回调
	options.FailedThresholdCallback = async (message) =>
	{
		try
		{
			string title = "【CAP事件总线】异常提醒  ";
			string msgId = message.Message?.Headers["cap-msg-id"]; //获取消息Id
			string msgName = message.Message?.Headers["cap-msg-name"]; //获取消息名称
			string msgSetTime = message.Message?.Headers["cap-senttime"]; //获取发送时间
			string body = $"CAP事件总线异常 - MsgId:[{msgId}], MsgName:[{msgName}], MsgSetTime:[{msgSetTime}], Msg:[{message.Message.Value}]";
			switch (message?.MessageType) //消息类型
			{
				case MessageType.Publish:
					title += "[Publish]";
					break;
				case MessageType.Subscribe:
					title += "[Subscribe]";
					break;
				default:
					break;
			};
			
			Console.WriteLine(title, body);
		}
		catch (Exception ex)
		{
			Console.WriteLine($"【CAP事件总线】异常提醒 发送失败! {ex.Message}");
		}
	};
});
```


### 简单使用

- 推送消息
```c#
public class PublishService
{
	private readonly ICapPublisher _capPublisher;
	
	public PublishService(ICapPublisher capPublisher)
	{
		_capPublisher = capPublisher;
	}

	/// <summary>
	/// 发布信息
	/// </summary>
	/// <param name="msg"></param>
	/// <returns></returns>
	public async Task PublishMessage(string msg)
	{
		await _capPublisher.PublishAsync("test_message", msg);
	}
}
```

- 消费消息
```c#
public class SubscribeService : ICapSubscribe
{
	public SubscribeService() { }

	/// <summary>
	/// 发送信息
	/// </summary>
	/// <param name="msg">消息</param>
	/// <returns></returns> 
	[CapSubscribe("test_message")]
	public async Task ConsumeMessage(string msg)
	{
		Console.WriteLine($"消费消息：{msg}")
	}
}
```


