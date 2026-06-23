<p align="center">
  <a href="../en/Redis.md"><strong>English</strong></a> ｜ <a href="../cn/Redis.md">简体中文</a>
</p>

[← Contents](../../README.md)

# StackExchange.Redis

## Introduction

`StackExchange.Redis` is a general-purpose `Redis` client library. It provides many methods for working with `Redis` and is popular on `GitHub`.

`GitHub`: [![GitHub Release](https://img.shields.io/github/v/release/StackExchange/StackExchange.Redis)](https://github.com/StackExchange/StackExchange.Redis)

`NuGet`: [![NuGet Version](https://img.shields.io/nuget/v/StackExchange.Redis)](https://www.nuget.org/packages/StackExchange.Redis/)

Documentation: [https://stackexchange.github.io/StackExchange.Redis/](https://stackexchange.github.io/StackExchange.Redis/)


## Connection Configuration

- Add the `NuGet` package
```c#
StackExchange.Redis
```

- Connection configuration class
```c#
public static class RedisConnector
{
	/// <summary>
	/// Lock object
	/// </summary>
	private static readonly object _lock = new();
	/// <summary>
	/// Connection instance
	/// </summary>
	private static ConnectionMultiplexer _instance = null;

	/// <summary>
	/// Get an instance
	/// </summary>
	/// <param name="Configuration">Configuration</param>
	/// <param name="connectString">Connection string</param>
	/// <returns></returns>
	public static ConnectionMultiplexer GetInstance(IConfiguration Configuration, string connectString = "")
	{
		try
		{
			if (_instance == null)
			{
				lock (_lock)
				{
					if (_instance == null || !_instance.IsConnected)
					{
						_instance = Connect(Configuration, connectString);
					}
				}
			}
		}
		catch(Exception ex)
		{
			Console.WriteLine($"Redis获取连接失败!{ex.Message}"); // Failed to get a Redis connection.
		}

		return _instance;
	}

	/// <summary>
	/// Get a Redis connection
	/// </summary>
	/// <param name="Configuration">Configuration</param>
	/// <param name="connectString">Connection string</param>
	/// <returns></returns>
	private static ConnectionMultiplexer Connect(IConfiguration Configuration,string connectString = "")
	{
		// Get the connection string from appsettings.json or use the supplied string
		connectString = string.IsNullOrEmpty(connectString) ? Configuration.GetValue<string>("Redis.ConnectionString") : connectString;

		// Connect to Redis
		var connect = ConnectionMultiplexer.Connect(connectString);

		// Register events
		connect.ConnectionFailed += MuxerConnectionFailed;
		connect.ConnectionRestored += MuxerConnectionRestored;
		connect.ErrorMessage += MuxerErrorMessage;
		connect.ConfigurationChanged += MuxerConfigurationChanged;
		connect.HashSlotMoved += MuxerHashSlotMoved;
		connect.InternalError += MuxerInternalError;

		return connect;
	}
}
```

- Events registered when connecting
```c#
/// <summary>
/// When the configuration changes
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerConfigurationChanged(object sender, EndPointEventArgs e)
{
	Console.WriteLine("Configuration changed: " + e.EndPoint);
}
/// <summary>
/// When an error occurs
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerErrorMessage(object sender, RedisErrorEventArgs e)
{
	Console.WriteLine("ErrorMessage: " + e.Message);
}
/// <summary>
/// Error before the connection is restored
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerConnectionRestored(object sender, ConnectionFailedEventArgs e)
{
	Console.WriteLine("ConnectionRestored: " + e.EndPoint);
}
/// <summary>
/// Connection failed. You won't receive this notification if reconnection succeeds.
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerConnectionFailed(object sender, ConnectionFailedEventArgs e)
{
	Console.WriteLine("重新连接：Endpoint failed: " + e.EndPoint + ", " + e.FailureType + (e.Exception == null ? "" : (", " + e.Exception.Message))); // Reconnecting: endpoint failed.
}
/// <summary>
/// Cluster changed
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerHashSlotMoved(object sender, HashSlotMovedEventArgs e)
{
	Console.WriteLine("HashSlotMoved:NewEndPoint" + e.NewEndPoint + ", OldEndPoint" + e.OldEndPoint);
}
/// <summary>
/// Redis library error
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerInternalError(object sender, InternalErrorEventArgs e)
{
	Console.WriteLine("InternalError:Message" + e.Exception.Message);
}
```

## Redis Helper

The `Redis` helper reduces the need to repeat configuration for each `Redis` operation.

- Create a `Redis` helper interface
```c#
public interface IRedisHelper
{
	Task<string> StringGetAsync(string key);
	Task<bool> StringSetAsync(string key, string value, TimeSpan? expiry = default, When when = When.Always);
}
```

- Create a `Redis` helper
```c#
public class RedisHelper : IRedisHelper
{
	/// <summary>
	/// Redis database number
	/// </summary>
	private readonly int DATABASE_NUM;
	/// <summary>
	/// Custom prefix
	/// </summary>
	private string CUSTOMER_PREFIX_KEY;

	/// <summary>
	/// Connection
	/// </summary>
	private readonly ConnectionMultiplexer _conn;
	/// <summary>
	/// Configuration
	/// </summary>
	private readonly IConfiguration _configuration;

	/// <summary>
	/// Constructor
	/// </summary>
	/// <param name="configuration"></param>
	public RedisHelper(IConfiguration configuration) : this(configuration, null, null) { }

	/// <summary>
	/// Constructor
	/// </summary>
	/// <param name="configuration">Configuration</param>
	/// <param name="dbNum">Database number</param>
	public RedisHelper(IConfiguration configuration, int dbNum) : this(configuration, null, dbNum) { }

	/// <summary>
	/// Constructor
	/// </summary>
	/// <param name="configuration">Configuration</param>
	/// <param name="connectString">Connection string</param>
	/// <param name="dbNum">Database number</param>
	public RedisHelper(IConfiguration configuration, string connectString, int? dbNum)
	{
		_configuration = configuration;

		_conn = RedisConnector.GetInstance(configuration, connectString);

		// Get values from appsettings.json
		DATABASE_NUM = dbNum ?? configuration.GetValue<int>("Redis.DbNum");
		CUSTOMER_PREFIX_KEY = configuration.GetValue<string>("Redis.PrefixKey");
	}
}
```

- Core methods
```c#
/// <summary>
/// Get the Redis database
/// </summary>
/// <returns></returns>
public IDatabase GetDatabase()
{
	return _conn.GetDatabase(DATABASE_NUM);
}
/// <summary>
/// Add a prefix
/// </summary>
/// <param name="oldKey"></param>
/// <returns></returns>
private string AddPrefixKey(string oldKey)
{
	//var prefixKey = CustomKey ?? RedisConnectionHelper.SysCustomKey;
	var prefixKey = string.IsNullOrEmpty(CUSTOMER_PREFIX_KEY) ? _configuration.GetValue<string>("Redis.PrefixKey") : CUSTOMER_PREFIX_KEY;
	return prefixKey + oldKey;
}
/// <summary>
/// Perform an operation with a return value
/// </summary>
/// <typeparam name="T"></typeparam>
/// <param name="func"></param>
/// <returns></returns>
private T Do<T>(Func<IDatabase, T> func)
{
	var database = _conn.GetDatabase(DATABASE_NUM);
	return func(database);
}
/// <summary>
/// Perform an operation without a return value
/// </summary>
/// <param name="func"></param>
private void Do(Action<IDatabase> func)
{
	var database = _conn.GetDatabase(DATABASE_NUM);
	func(database);
}
```

- Example database operations

You can wrap more methods from the `StackExchange.Redis` documentation to make them easier and faster to use.
```c#
/// <summary>
/// Get the value of one key
/// </summary>
/// <param name="key">Redis Key</param>
/// <returns></returns>
public async Task<string> StringGetAsync(string key)
{
	key = AddPrefixKey(key);
	return await Do(db => db.StringGetAsync(key));
}

/// <summary>
/// Save one key-value pair
/// </summary>
/// <param name="key">Redis Key</param>
/// <param name="value">Value to save</param>
/// <param name="expiry">Expiration time</param>
/// <returns></returns>
public async Task<bool> StringSetAsync(string key, string value, TimeSpan? expiry = default, When when = When.Always)
{
	key = AddPrefixKey(key);
	return await Do(db => db.StringSetAsync(key, value, expiry, when));
}
```

## Usage

- Dependency injection
```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

// Register the Redis helper as a singleton to use it
builder.Services.AddSingleton<IRedisHelper, RedisHelper>();
```

- Example
```c#
public class TestService : ITestService
{
	/// <summary>
	/// Redis
	/// </summary>
	private readonly IRedisHelper _redis;

	/// <summary>
	/// Constructor
	/// </summary>
	public TestService(IRedisHelper redis)
	{
		_redis = redis; // Constructor injection
	}

	/// <summary>
	/// Get
	/// </summary>
	/// <returns></returns>
	public async Task<string> Get()
	{
		return await _redis.StringGetAsync("test");
	}

	/// <summary>
	/// Set
	/// </summary>
	/// <param name="value">Value to store</param>
	/// <returns></returns>
	public async Task<bool> Set(string value)
	{
		return await _redis.StringSetAsync("test", value);
	}
}
```
