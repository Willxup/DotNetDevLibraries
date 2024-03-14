
🚧**装修中...**

# Hangfire

## 介绍

`Hangfire`是一个开源的`.net`任务调用框架，它提供了内置集成化的控制台，可以直观明了的查看作业调度情况，并且`Hangfire`不需要依赖于单独的应用程序执行。

`Github`：[![GitHub Release](https://img.shields.io/github/v/release/HangfireIO/Hangfire)](https://github.com/HangfireIO/Hangfire)

`Nuget`：[![NuGet Version](https://img.shields.io/nuget/v/Hangfire.AspNetCore)](https://www.nuget.org/packages/Hangfire.AspNetCore)

文档：[https://docs.hangfire.io/en/latest/](https://docs.hangfire.io/en/latest/)


## 快速上手

### Nuget包

```c#
Hangfire.AspNetCore
Hangfire.Redis.StackExchange //redis持久化
Hangfire.Dashboard.BasicAuthorization //登录授权
```

### 服务注入
```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

// hangfire
builder.Services.AddHangfire(config =>
config
	.SetDataCompatibilityLevel(CompatibilityLevel.Version_180)
	.UseSimpleAssemblyNameTypeSerializer()
	.UseRecommendedSerializerSettings()
	.UseRedisStorage(ConnectionMultiplexer.Connect(configuration.GetValue<string>("Redis.ConnectionString")),
		new RedisStorageOptions
		{
			Db = configuration.GetValue<int>("Redis.DbNum"),
			Prefix = "FastAdminAPI_Hangfire:"
		}).WithJobExpirationTimeout(TimeSpan.FromDays(7))
);
//用于启动hangfire
builder.Services.AddHangfireServer();
```

### 面板设置
```c#
WebApplication app = builder.Build();

//启用Hangfire控制台
app.UseHangfireDashboard("/dashboard", new DashboardOptions
{
	DashboardTitle = "FastAdminAPI Tasks Dashboard", //页面标题
	AppPath = "/dashboard",//返回时跳转的地址
	DisplayStorageConnectionString = false,//是否显示数据库连接信息
	DefaultRecordsPerPage = 50, //默认每页显示数据条数
	Authorization = new[] { new BasicAuthAuthorizationFilter(new BasicAuthAuthorizationFilterOptions
	{
		RequireSsl = false, //是否启用ssl验证，即https
		SslRedirect = false,
		LoginCaseSensitive = true,
		Users = new []
		{
			new BasicAuthAuthorizationUser
			{
				Login = "fastadminapi",
				PasswordClear =  "123456"
			}
		}
	})}
});

app.UseEndpoints(endpoints => { endpoints.MapHangfireDashboard(); });
```

### 任务配置

- 任务
```c#
public abstract class BaseTask
{
	/// <summary>
	/// 子类实现的要做的事情
	/// </summary>
	public abstract Task Run();
}

public class TestTask : BaseTask
{
	public override async Task Run()
	{
		//do something
	}
}
```

- 创建任务
```c#
//设置时区
RecurringJobOptions options = new()
{
	TimeZone = TimeZoneInfo.Local,
	MisfireHandling = MisfireHandlingMode.Relaxed
};

//实例化任务，只要继承了BaseTask的类就可以实例化
BaseTask task = Activator.CreateInstance(typeof(TestTask)) as BaseTask;


//添加定时任务 
// 参数：任务Id, 执行的方法, cron表达式, 任务设置
RecurringJob.AddOrUpdate("Task.TestTask", () => task.Run(), () => Cron.Daily(1,1), options);

app.Run();
```

启动项目即可按照设置的`cron`表达式，进行定时任务了。