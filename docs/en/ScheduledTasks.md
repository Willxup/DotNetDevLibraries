<p align="center">
  <a href="../en/ScheduledTasks.md"><strong>English</strong></a> ｜ <a href="../cn/ScheduledTasks.md">简体中文</a>
</p>

[← Contents](../../README.md)

# Hangfire

## Introduction

`Hangfire` is an open-source .NET framework for background jobs. Its built-in dashboard makes it easy to see scheduled jobs, and it does not need a separate application to run them.

`GitHub`: [![GitHub Release](https://img.shields.io/github/v/release/HangfireIO/Hangfire)](https://github.com/HangfireIO/Hangfire)

`NuGet`: [![NuGet Version](https://img.shields.io/nuget/v/Hangfire.AspNetCore)](https://www.nuget.org/packages/Hangfire.AspNetCore)

Documentation: [https://docs.hangfire.io/en/latest/](https://docs.hangfire.io/en/latest/)


## Quick Start

### NuGet Packages

```c#
Hangfire.AspNetCore
Hangfire.Redis.StackExchange // Redis persistence
Hangfire.Dashboard.BasicAuthorization // Login authorization
```

### Service Registration
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
		}).WithJobExpirationTimeout(TimeSpan.FromDays(1))
);
// Start Hangfire
builder.Services.AddHangfireServer();
```

### Dashboard Settings
```c#
WebApplication app = builder.Build();

// Enable the Hangfire dashboard
app.UseHangfireDashboard("/dashboard", new DashboardOptions
{
	DashboardTitle = "FastAdminAPI Tasks Dashboard", // Page title
	AppPath = "/dashboard",// Address to return to
	DisplayStorageConnectionString = false,// Whether to show database connection information
	DefaultRecordsPerPage = 50, // Default number of records per page
	Authorization = new[] { new BasicAuthAuthorizationFilter(new BasicAuthAuthorizationFilterOptions
	{
		RequireSsl = false, // Whether to require SSL (HTTPS)
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

### Job Configuration

- Job
```c#
public abstract class BaseScheduleJob
{
	/// <summary>
	/// Work to be implemented by subclasses
	/// </summary>
	public abstract Task Run();
}

public class TestSchedule : BaseScheduleJob
{
	public override async Task Run()
	{
		//do something
	}
}
```

- Create a recurring job
```c#
// Set the time zone
RecurringJobOptions options = new()
{
	TimeZone = TimeZoneInfo.Local,
	MisfireHandling = MisfireHandlingMode.Relaxed
};

// Create a job instance; classes that inherit BaseTask can be instantiated
BaseTask task = Activator.CreateInstance(typeof(TestSchedule)) as BaseTask;


// Add a recurring job
// Parameters: job ID, method to run, cron expression, job options
RecurringJob.AddOrUpdate("Task.TestTask", () => task.Run(), () => Cron.Daily(1,1), options);

app.Run();
```

Start the project to run the recurring job on the configured `cron` schedule.
