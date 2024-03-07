
🚧**装修中...**

# 目录

- [官网网站](#官网网站)
- [开发小技巧](#开发小技巧)

# 官网网站

官方文档是最正确的文档，虽然微软的机翻实在不怎么行，不过还是建议尽量看官方文档，解决不了的再去查其他资料。

- [c#官网文档](https://learn.microsoft.com/zh-cn/dotnet/csharp/)
- [c#官方在线代码运行](https://learn.microsoft.com/zh-cn/dotnet/csharp/tour-of-csharp/#code-try-0)
- [.net官方文档](https://learn.microsoft.com/zh-cn/dotnet/fundamentals/)
- [.net官方下载](https://dotnet.microsoft.com/zh-cn/download/dotnet)

# 开发小技巧

这里总结了很多开发中能实际用到的一些小技巧，可以大大简化代码量，提升效率。

## 语法糖

> 语法糖需要根据`c#`版本来确实是否可以使用，一般情况下`c# 8.0`及以上的`C#`版本都已支持。

- 对象判空
```c#
// 判断对象是否为空，为空抛出异常
if(obj == null) throw new NullReferenceException();

// 简化的语法糖
obj ?? throw new NullReferenceException();
```

- 对象为空赋值
```c#
// 判断对象 为空 的情况下再赋新值
// 如果对象 不为空 不进行赋值
if(obj == null)
{
	obj = new object();
}

// 简化的语法糖
obj ??= new object();
```

- 可空类型判空并赋值
```c#
// 可空类型
int? nums = null;

// 判断值是否为空，并进行不同的赋值
if(nums == null)
{
	result = -1;
} 
else 
{
	result = nums;
}

// 简化的语法糖
int result = nums ?? -1;
```

- 减少空引用
```c#
// 判断数组或list不能null且有元素
if(list != null && list.Count > 0)

// 简化的语法糖 当list为null时，将直接返回false
if(list?.Count > 0)

// 同样可运用在赋值时，如果obj为null，将不会取obj.text的值，而是将会为text赋值null
string text = obj?.text;
```

- 判断参数类型并转换类型+校验
```c#
// 判断value是否为 string 类型，如果value是 string 类型
// 那么将value转换为 string 类型，并赋值给 stringValue
// 再判断 stringValue是否不为Null或空
if(value is string stringValue && !string.IsNullOrEmpty(stringValue))
```

- 切片操作
```c#
// **以下所有[]中的数字都代表索引**
// **如果是范围索引，且声明结束索引，那么都将不包含结束索引的值**

// 数组例子
string[] arr = new string[] { "10", "20", "30", "40", "50", "60", "70", "80", "90", "100" };

// 获取最后一个元素
string str = arr[^1];

// 获取前3个元素，从索引0开始 到 索引3(不包含)：["10","20","30"]
// 可省略索引0，从开始 到 索引3(不包含)
// string[] strs = arr[..3];
string[] strs1 = arr[0..3];

// 获取后3个元素，从倒数第3个元素开始 到 最后：["80", "90", "100"]
// 最后一位索引被省略 string[] strs21 = arr[^3..^0];
// ^0 倒数第0个元素是不存在的
string[] strs2 = arr[^3..];

// 指定获取 正向 某一段元素
// 从索引3开始 到 索引7(不包含)：["40", "50", "60", "70"]
string[] strs3 = arr[3..7];

// 指定获取 反向 某一段元素
// 倒数第4个元素开始 到 倒数第2个元素(不包含)：["70","80"]
string[] strs4 = arr[^4..^2];
```

- Switch
```c#
public string GetNums(int num)
{
	// 使用这种方式的switch时，要求返回类型统一
	string str = num switch
	{
		1 => "num的值是1",
		2 => "num的值是2",
		3 => "num的值是3",
		4 => "num的值是4",
		_ => "其他"
	};

	return str;
}
```

