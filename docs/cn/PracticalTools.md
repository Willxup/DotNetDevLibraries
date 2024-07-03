# 目录

- [小工具源码](#小工具)
- [特性源码](#特性)
- [日志源码](#日志)

# 小工具

- Guid
```c#
/// <summary>
/// 将一个标准的GUID转换成短的字符串如：3d4ebc5f5f2c4ede
/// </summary>
/// <returns></returns>
public static string GenerateShortGuid()
{
	long i = 1;
	foreach (byte b in Guid.NewGuid().ToByteArray())
	{
		i *= b + 1;
	}
	return string.Format("{0:x}", i - DateTime.Now.Ticks);
}
/// <summary>
/// 获得一个19位长的序列,8346734568923542345
/// </summary>
/// <returns></returns>
public static string GenerateIntegerGuid()
{
	byte[] buffer = Guid.NewGuid().ToByteArray();
	return BitConverter.ToInt64(buffer, 0).ToString();
}
```

- 时间转换时间戳
```c#
/// <summary>
/// 时间戳类型
/// </summary>
public enum TimeStampType 
{
	/// <summary>
	/// 秒
	/// </summary>
	Seconds = 1,
	/// <summary>
	/// 毫秒
	/// </summary>
	Milliseconds = 2 
}
/// <summary>  
/// DateTime时间格式转换为时间戳 
/// </summary>  
/// <param name="time"> DateTime时间格式</param>  
/// <returns>Unix时间戳格式(默认毫秒)</returns>  
public static string ConvertDateToTimeStamp(DateTime time, TimeStampType timeStampType = TimeStampType.Milliseconds)
{
	//DateTime startTime = TimeZone.CurrentTimeZone.ToLocalTime(new DateTime(1970, 1, 1));
	//DateTime startTime = TimeZoneInfo.ConvertTime(new DateTime(1970, 1, 1), TimeZoneInfo.Local);
	DateTime startTime = new DateTime(1970, 1, 1).ToLocalTime();
	var timeSpan = time - startTime;
	double timeStamp = 0;
	switch (timeStampType)
	{
		case TimeStampType.Seconds:
			timeStamp = timeSpan.TotalSeconds;
			break;
		case TimeStampType.Milliseconds:
			timeStamp = timeSpan.TotalMilliseconds;
			break;
	}
	return timeStamp.ToString();
}
/// <summary>
/// DateTime时间格式转换为时间戳 
/// </summary>
/// <param name="date">DateTime时间格式</param>
/// <returns></returns>
public static double ConvertDateToTimeStamp(DateTime date)
{
	return (date.ToUniversalTime().Ticks - 621355968000000000) / 10000000;
}
/// <summary>
/// 时间戳转DateTime
/// </summary>
/// <param name="timestamp">时间戳</param>
/// <param name="timeStampType">时间戳类型(默认毫秒)</param>
/// <returns></returns>
public static DateTime? ConvertTimeStampToDate(string timestamp, TimeStampType timeStampType = TimeStampType.Milliseconds)
{
	DateTime startTime = new DateTime(1970, 1, 1, 0, 0, 0, 0);
	DateTime? time = null;
	double stamp = -1;
	try
	{
		stamp = Convert.ToDouble(timestamp);
		if (stamp >= 0)
		{
			switch (timeStampType)
			{
				case TimeStampType.Seconds:
					time = startTime.AddSeconds(stamp).ToLocalTime();
					break;
				case TimeStampType.Milliseconds:
					time = startTime.AddMilliseconds(stamp).ToLocalTime();
					break;
			}
		}
	}
	catch (Exception)
	{
		//返回null
	}
	return time;
}
```

- 时间格式化为字符串
```c#
/// <summary>
/// 可空时间类型格式化为字符串
/// </summary>
/// <param name="datetime">时间</param>
/// <param name="format"></param>
/// <returns></returns>
public static string ToFormattedString(this DateTime? datetime, string format = "yyyy-MM-dd HH:mm:ss", DateTime? defaultValue = null)
{
	if(datetime != null)
	{
		return ((DateTime)datetime).ToFormattedString(format);
	}
	return defaultValue != null ? ((DateTime)defaultValue).ToFormattedString(format) : DateTime.MinValue.ToFormattedString(format);
}
/// <summary>
/// 时间类型格式化为字符串
/// </summary>
/// <param name="datetime"></param>
/// <param name="format"></param>
/// <returns></returns>
public static string ToFormattedString(this DateTime datetime, string format = "yyyy-MM-dd HH:mm:ss")
{
	return datetime.ToString(format, DateTimeFormatInfo.InvariantInfo);
}
```

- 对象判空(反射)
```c#
/// <summary>
/// 判断对象是否为空
/// </summary>
/// <typeparam name="T"></typeparam>
/// <param name="data"></param>
/// <returns></returns>
public static bool ObjectIsNullOrEmpty<T>(T data)
	where T : class, new()
{
	//是否为空 默认为空
	bool isNullOrEmpty = true;

	//获取类型的所有属性
	var properties = data.GetType().GetProperties();
	if (properties?.Length > 0)
	{
		foreach (var property in properties)
		{
			//获取属性的值
			var value = property.GetValue(data);

			//判断值是否为null
			if (value is null)
			{
				continue;
			}
			//判断值是否为字符串且字符串为空
			else if (value is string str && string.IsNullOrWhiteSpace(str))
			{
				continue;
			}
			//其他情况
			else
			{
				isNullOrEmpty = false;
				break;
			}
		}
	}

	return isNullOrEmpty;
}
```
# 特性

- 枚举校验
```c#
[AttributeUsage(AttributeTargets.Property | AttributeTargets.Field | AttributeTargets.Parameter, AllowMultiple = false)]
public class EnumCheckAttribute : RequiredAttribute
{
	/// <summary>
	/// 枚举类型
	/// </summary>
	private readonly Type _enumType;
	/// <summary>
	/// 是否允许为空
	/// </summary>
	private readonly bool _IsAllowEmpty;
	
	/// <summary>
	/// 构造
	/// </summary>
	/// <param name="enumType">枚举类</param>
	/// <param name="IsAllowEmpty">是否允许为空 默认否</param>
	public EnumCheckAttribute(Type enumType, bool IsAllowEmpty = false)
	{
		_enumType = enumType;
		_IsAllowEmpty = IsAllowEmpty;
	}

	/// <summary>
	/// 校验
	/// </summary>
	/// <param name="value"></param>
	/// <returns></returns>
	public override bool IsValid(object value)
	{
		if (value is null)
		{
			return _IsAllowEmpty;
		}
		if (!_enumType.IsEnum)
		{
			ErrorMessage = "该类型不是枚举类型!";
			return false;
		}
		try
		{
			if (Enum.IsDefined(_enumType, Convert.ToInt32(value)))
			{
				return true;
			}
			else
			{
				ErrorMessage = $"请输入正确的值!";
				return false;
			}
		}
		catch (InvalidOperationException)
		{
			ErrorMessage = $"不支持的传参类型!";
			return false;
		}
		catch (Exception ex)
		{
			ErrorMessage = $"参数类型错误, {ex.Message}";
			return false;
		}
	}
}
```

- 手机号校验
```c#
[AttributeUsage(AttributeTargets.Property | AttributeTargets.Field, AllowMultiple = false)]
public class PhoneCheckAttribute : ValidationAttribute
{
	/// <summary>
	/// 是否允许为空
	/// </summary>
	private readonly bool _isAllowEmpty;

	/// <summary>
	/// 构造
	/// </summary>
	/// <param name="isAllowEmpty">是否允许为空(默认false)</param>
	/// <param name="ErrorMessage">错误信息</param>
	public PhoneCheckAttribute(bool isAllowEmpty = false, string ErrorMessage = "手机号码输入错误")
	{
		base.ErrorMessage = ErrorMessage;
		_isAllowEmpty = isAllowEmpty;
	}

	/// <summary>
	/// 校验
	/// </summary>
	/// <param name="value"></param>
	/// <returns></returns>
	/// <exception cref="UserOperationException"></exception>
	public override bool IsValid(object value)
	{
		if (value is null)
		{
			//不允许为空抛出异常
			if (!_isAllowEmpty)
				throw new UserOperationException(ErrorMessage);
			else
				return true;
		}
		Type type = value.GetType();
		if (type == typeof(string))
		{
			string phone = value.ToString();
			if (phone.Length != 11)
				throw new UserOperationException("请输入11位正确的手机号码！");
			if (!Regex.IsMatch(phone, @"^1\d{10}$"))
				throw new UserOperationException("手机号码格式错误,请检查！");

		}
		else
			throw new UserOperationException("数据类型错误");
		return true;
	}
}
```

# 日志

```c#
/// <summary>
/// 高效日志
/// </summary>
public static class LogLockHelper
{
	/// <summary>
	/// 读写锁
	/// </summary>
	private static readonly ReaderWriterLockSlim _lock = new();

	/// <summary>
	/// 写入次数
	/// </summary>
	private static int _writedCount = 0;
	/// <summary>
	/// 失败次数
	/// </summary>
	private static int _failedCount = 0;

	/// <summary>
	/// 写入日志
	/// </summary>
	/// <param name="fileName"></param>
	/// <param name="dataParas"></param>
	public static void WriteLog(string fileName, params string[] dataParas)
	{
		try
		{
			_lock.EnterWriteLock();

			var path = Path.Combine(Directory.GetCurrentDirectory() + "/logs/");
			if (!Directory.Exists(path))
			{
				Directory.CreateDirectory(path);
			}
			if (fileName == "RequestResponseLog")
			{
				fileName = DateTime.Now.ToString("yyyyMMdd") + fileName;
			}
			var name = fileName + ".log";

			string logFilePath = Path.Combine(path + name);

			var now = DateTime.Now;
			var logContent = (
				"--------------------------------\r\n" +
				DateTime.Now + "|\r\n" +
				string.Join("\r\n", dataParas) + "\r\n"
				);

			File.AppendAllText(logFilePath, logContent);

			_writedCount++;
		}
		catch (Exception e)
		{
			Console.Write(e.Message);
			_failedCount++;
		}
		finally
		{
			_lock.ExitWriteLock();
		}
	}

	/// <summary>
	/// 读取日志
	/// </summary>
	/// <param name="path">日志路径</param>
	/// <param name="encode">编码(默认UTF8)</param>
	/// <returns></returns>
	public static string ReadLog(string path, Encoding encode = null)
	{
		string log = "";
		try
		{
			_lock.EnterReadLock();

			if (!File.Exists(path))
			{
				log = null;
			}
			else
			{
				StreamReader f2 = new(path, encode ?? Encoding.UTF8);
				log = f2.ReadToEnd();
				f2.Close();
				f2.Dispose();
			}
		}
		catch (Exception)
		{
			_failedCount++;
		}
		finally
		{
			_lock.ExitReadLock();
		}
		return log;
	}
	
	/// <summary>
	/// 获取写入次数
	/// </summary>
	/// <returns></returns>
	public static int GetWritedCount()
	{
		return _writedCount;
	}
	/// <summary>
	/// 获取失败次数
	/// </summary>
	/// <returns></returns>
	public static int GetFailedCount()
	{
		return _failedCount;
	}
}
```