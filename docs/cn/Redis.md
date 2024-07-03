# StackExchange.Redis

## 介绍

`StackExchange.Redis`是一个`Redis`的通用客户端类库，它封装了操作`Redis`的很多方法，可以方便的使用。它在`github`上也是非常受欢迎的类库。

`Github`：[![GitHub Release](https://img.shields.io/github/v/release/StackExchange/StackExchange.Redis)](https://github.com/StackExchange/StackExchange.Redis)

`Nuget`：[![NuGet Version](https://img.shields.io/nuget/v/StackExchange.Redis)](https://www.nuget.org/packages/StackExchange.Redis/)

文档：[https://stackexchange.github.io/StackExchange.Redis/](https://stackexchange.github.io/StackExchange.Redis/)


## 连接配置

- 引入`nuget`
```c#
StackExchange.Redis
```

- 连接配置类
```c#
public static class RedisConnector
{
	/// <summary>
	/// 对象锁
	/// </summary>
	private static readonly object _lock = new();
	/// <summary>
	/// 连接实例
	/// </summary>
	private static ConnectionMultiplexer _instance = null;

	/// <summary>
	/// 获取实例
	/// </summary>
	/// <param name="Configuration">配置</param>
	/// <param name="connectString">连接字符串</param>
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
			Console.WriteLine($"Redis获取连接失败!{ex.Message}");
		}
			
		return _instance;
	}

	/// <summary>
	/// 获取Redis连接
	/// </summary>
	/// <param name="Configuration">配置</param>
	/// <param name="connectString">连接字符串</param>
	/// <returns></returns>
	private static ConnectionMultiplexer Connect(IConfiguration Configuration,string connectString = "")
	{
		//连接字符串，从appsettings.json中获取，或使用指定连接字符串
		connectString = string.IsNullOrEmpty(connectString) ? Configuration.GetValue<string>("Redis.ConnectionString") : connectString;
		
		//连接Redis
		var connect = ConnectionMultiplexer.Connect(connectString);

		//注册事件
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

- 连接时注册的事件
```c#
/// <summary>
/// 配置更改时
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerConfigurationChanged(object sender, EndPointEventArgs e)
{
	Console.WriteLine("Configuration changed: " + e.EndPoint);
}
/// <summary>
/// 发生错误时
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerErrorMessage(object sender, RedisErrorEventArgs e)
{
	Console.WriteLine("ErrorMessage: " + e.Message);
}
/// <summary>
/// 重新建立连接之前的错误
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerConnectionRestored(object sender, ConnectionFailedEventArgs e)
{
	Console.WriteLine("ConnectionRestored: " + e.EndPoint);
}
/// <summary>
/// 连接失败, 如果重新连接成功你将不会收到这个通知
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerConnectionFailed(object sender, ConnectionFailedEventArgs e)
{
	Console.WriteLine("重新连接：Endpoint failed: " + e.EndPoint + ", " + e.FailureType + (e.Exception == null ? "" : (", " + e.Exception.Message)));
}
/// <summary>
/// 更改集群
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerHashSlotMoved(object sender, HashSlotMovedEventArgs e)
{
	Console.WriteLine("HashSlotMoved:NewEndPoint" + e.NewEndPoint + ", OldEndPoint" + e.OldEndPoint);
}
/// <summary>
/// redis类库错误
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private static void MuxerInternalError(object sender, InternalErrorEventArgs e)
{
	Console.WriteLine("InternalError:Message" + e.Exception.Message);
}
```

## Redis帮助类

`Redis`帮助类，是为了减少每次操作`Redis`时，需要重复进行配置的问题。

- 创建`Redis`帮助类接口
```c#
public interface IRedisHelper
{

}
```

- 创建`Redis`帮助类
```c#
public class RedisHelper : IRedisHelper
{
	/// <summary>
	/// redis数据库编号
	/// </summary>
	private readonly int DATABASE_NUM;
	/// <summary>
	/// 自定义前缀
	/// </summary>
	private string CUSTOMER_PREFIX_KEY;

	/// <summary>
	/// 连接
	/// </summary>
	private readonly ConnectionMultiplexer _conn;
	/// <summary>
	/// 配置
	/// </summary>
	private readonly IConfiguration _configuration;

	/// <summary>
	/// 构造
	/// </summary>
	/// <param name="configuration"></param>
	public RedisHelper(IConfiguration configuration) : this(configuration, null, null) { }

	/// <summary>
	/// 构造
	/// </summary>
	/// <param name="configuration">配置</param>
	/// <param name="dbNum">数据库编号</param>
	public RedisHelper(IConfiguration configuration, int dbNum) : this(configuration, null, dbNum) { }

	/// <summary>
	/// 构造
	/// </summary>
	/// <param name="configuration">配置</param>
	/// <param name="connectString">连接字符串</param>
	/// <param name="dbNum">数据库编号</param>
	public RedisHelper(IConfiguration configuration, string connectString, int? dbNum)
	{
		_configuration = configuration;

		_conn = RedisConnector.GetInstance(configuration, connectString);

		//从appsettings.json中获取
		DATABASE_NUM = dbNum ?? configuration.GetValue<int>("Redis.DbNum");
		CUSTOMER_PREFIX_KEY = configuration.GetValue<string>("Redis.PrefixKey");
	}
}
```

- 核心方法
```c#
/// <summary>
/// 获取Redis数据库
/// </summary>
/// <returns></returns>
public IDatabase GetDatabase()
{
	return _conn.GetDatabase(DATABASE_NUM);
}
/// <summary>
/// 添加前缀
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
/// 操作(带返回值)
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
/// 操作(不带返回值)
/// </summary>
/// <param name="func"></param>
private void Do(Action<IDatabase> func)
{
	var database = _conn.GetDatabase(DATABASE_NUM);
	func(database);
}
```

- 数据库操作方法实例

这些方法可以参考`stackexchange.redis`文档的方法进行进一步的封装，这样在使用时会更加简单快捷。
```c#
/// <summary>
/// 获取单个key的值
/// </summary>
/// <param name="key">Redis Key</param>
/// <returns></returns>
public async Task<string> StringGetAsync(string key)
{
	key = AddPrefixKey(key);
	return await Do(db => db.StringGetAsync(key));
}

/// <summary>
/// 保存单个key value
/// </summary>
/// <param name="key">Redis Key</param>
/// <param name="value">保存的值</param>
/// <param name="expiry">过期时间</param>
/// <returns></returns>
public async Task<bool> StringSetAsync(string key, string value, TimeSpan? expiry = default, When when = When.Always)
{
	key = AddPrefixKey(key);
	return await Do(db => db.StringSetAsync(key, value, expiry, when));
}
```

## 使用

- 依赖注入
```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

// 单例注入Redis帮助类即可使用
builder.Services.AddSingleton<IRedisHelper, RedisHelper>();
```

- 实例
```c#
public class TestService : ITestService
{
	/// <summary>
	/// Redis
	/// </summary>
	private readonly IRedisHelper _redis;

	/// <summary>
	/// 构造
	/// </summary>
	public TestService(IRedisHelper redis)
	{
		_redis = redis; //构造注入
	}

	/// <summary>
	/// 获取
	/// </summary>
	/// <returns></returns>
	public async Task<string> Get()
	{
		return await _redis.StringGetAsync("test");
	}
	
	/// <summary>
	/// 设置
	/// </summary>
	/// <param name="value">存储的值</param>
	/// <returns></returns>
	public async Task<bool> Set(string value)
	{
		return await _redis.StringSetAsync("test", value);
	}
}
```