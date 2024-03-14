
🚧**装修中...**

# Refit

## 介绍

`Refit`是一个基于`REST`的`HTTP`请求库。它可以让我们在进行网络请求时无需编写`url`，直接像本地方法一样去调用。在`Java`中有`Springcloud`的`OpenFeign`，我一直在寻找功能类似的`C#`库，直到发现了`Refit`项目。

`Github`：[![GitHub Release](https://img.shields.io/github/v/release/reactiveui/refit)](https://github.com/reactiveui/refit)

`Nuget`：[![NuGet Version](https://img.shields.io/nuget/v/Refit)](https://www.nuget.org/packages/Refit/)

## 快速上手

### Nuget包

```c#
Refit.HttpClientFactory
Refit.Newtonsoft.Json
```

### IOC注入

```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

//这里注入了一个ITestApi类型的接口，这个接口在注入前需要创建(创建方式请往下看)
builder.Services.AddRefitClient<ITestApi>(new RefitSettings())
				.ConfigureHttpClient(c =>
				{
					//请求的基础地址，域名
					c.BaseAddress = new Uri("http://127.0.0.1:9001");
					//设置超时时间
					c.Timeout = TimeSpan.FromSeconds(60);
				});
```


### 简单使用

- Refit接口
```c#
public interface ITestApi
{
	/// <summary>
	/// 查询
	/// </summary>
	/// <param name="id"></param>
	/// <returns></returns>
	[Get("/api/Test/Get")]
	Task<string> Get([Query] long id);

	/// <summary>
	/// 插入
	/// </summary>
	/// <param name="dto"></param>
	/// <returns></returns>
	[Post("/api/Test/Add")]
	Task<bool> Add([Body] DtoModel dto);
}
```
只需要在接口上配置请求方式，对参数进行设置，以及对路由进行设置即可。

- 调用
```c#
public class TastService
{
	private readonly ITestApi _testApi;

	public TastService(ITestApi testApi)
	{
		_testApi = testApi
	}

	/// <summary>
	/// 查询
	/// </summary>
	/// <returns></returns>
	public async Task<string> Get()
	{
		//业务逻辑...

		//调用Get请求，http://127.0.0.1:9001/api/Test/Get?id=111
		return await _testApi.Get(111); 
	}

	/// <summary>
	/// 查询
	/// </summary>
	/// <returns></returns>
	public async Task<bool> Add()
	{
		DtoModel dto = new();
		//业务逻辑...

		//调用Post请求，http://127.0.0.1:9001/api/Test/Add
		return await _testApi.Add(dto);
	}
}
```
将`ITestApi`接口从构造注入进来，在需要调用的地方直接调用即可。使用`Refit`后，再也无需写一大堆请求地址，做一系列转换了，只需要建立接口，直接本地调用即可。