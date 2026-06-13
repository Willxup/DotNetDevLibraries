<p align="center">
  <a href="../en/MessageQueue.md"><strong>English</strong></a> ｜ <a href="../cn/MessageQueue.md">简体中文</a>
</p>

[← Contents](../../README.md)

# CAP

## Introduction

CAP is an event bus and a framework designed to help address distributed transactions in microservice or SOA systems. It helps build microservices that are scalable, reliable, and easy to change. Besides acting as a message queue, CAP can also help synchronize distributed transactions.

- Supported databases: `MySQL`, `SQL Server`, `PostgreSQL`, and `MongoDB`

- Supported transports include `Kafka`, `RabbitMQ`, and `Redis`

`GitHub`: [![GitHub Release](https://img.shields.io/github/v/release/dotnetcore/CAP)](https://github.com/dotnetcore/CAP)

`NuGet`: [![NuGet Version](https://img.shields.io/nuget/v/DotNetCore.CAP)](https://www.nuget.org/packages/DotNetCore.CAP)

Documentation: [https://cap.dotnetcore.xyz/](https://cap.dotnetcore.xyz/)

## Quick Start

### NuGet Packages

```c#
DotNetCore.CAP
DotNetCore.CAP.MySql // MySQL example
DotNetCore.CAP.RedisStreams // Redis example

DotNetCore.CAP.Dashboard // Dashboard
```


### Dependency Injection

```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

builder.Services.AddCap(options =>
{
	// Use MySQL for storage
	options.UseMySql(config =>
	{
		// Get the MySQL connection string from appsettings
		config.ConnectionString = configuration.GetValue<string>("Database.ConnectionString");

		// Prefix for tables created by CAP
		config.TableNamePrefix = "SYS_CAP";
	});

	// Use Redis Streams for message transport
	options.UseRedis(config =>
	{
		// Get the Redis connection string from appsettings
		ConfigurationOptions opt = ConfigurationOptions.Parse(configuration.GetValue<string>("Redis.ConnectionString"));
		// Default database
		opt.DefaultDatabase = configuration.GetValue<int>("Redis.DbNum");

		config.Configuration = opt;
	});


	// Set the number of retries after failure
	options.FailedRetryCount = 5;

	// Set the interval before the next attempt after failure (seconds)
	options.FailedRetryInterval = 60;

	// Set how long successful messages stay in the database (seconds); data is cleaned up regularly for performance.
	options.SucceedMessageExpiredAfter = 604800; // 7 days

	// Set how long failed messages stay in the database (seconds)
	options.FailedMessageExpiredAfter = 604800; // 7 days

	// Group prefix
	options.GroupNamePrefix = "FastAdminAPI";
	// Default group name
	options.DefaultGroupName = "DefultGroup";

	//// Topic prefix
	//x.TopicNamePrefix = "Topic.";

	// Callback when failures exceed the retry limit
	options.FailedThresholdCallback = async (message) =>
	{
		try
		{
			string title = "【CAP事件总线】异常提醒  "; // CAP event bus error alert.
			string msgId = message.Message?.Headers["cap-msg-id"]; // Get the message ID
			string msgName = message.Message?.Headers["cap-msg-name"]; // Get the message name
			string msgSetTime = message.Message?.Headers["cap-senttime"]; // Get the sent time
			string body = $"CAP事件总线异常 - MsgId:[{msgId}], MsgName:[{msgName}], MsgSetTime:[{msgSetTime}], Msg:[{message.Message.Value}]"; // CAP event bus error.
			switch (message?.MessageType) // Message type
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
			Console.WriteLine($"【CAP事件总线】异常提醒 发送失败! {ex.Message}"); // Failed to send the CAP event bus error alert.
		}
	};
});
```


### Basic Usage

- Publish Messages
```c#
public class PublishService
{
	private readonly ICapPublisher _capPublisher;

	public PublishService(ICapPublisher capPublisher)
	{
		_capPublisher = capPublisher;
	}

	/// <summary>
	/// Publish a message
	/// </summary>
	/// <param name="msg"></param>
	/// <returns></returns>
	public async Task PublishMessage(string msg)
	{
		await _capPublisher.PublishAsync("test_message", msg);
	}
}
```

- Consume Messages
```c#
public class SubscribeService : ICapSubscribe
{
	public SubscribeService() { }

	/// <summary>
	/// Consume a message
	/// </summary>
	/// <param name="msg">Message</param>
	/// <returns></returns>
	[CapSubscribe("test_message")]
	public async Task ConsumeMessage(string msg)
	{
		Console.WriteLine($"消费消息：{msg}"); // Consuming message.
	}
}
```

