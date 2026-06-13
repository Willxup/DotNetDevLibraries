<p align="center">
  <a href="../en/Network.md"><strong>English</strong></a> ｜ <a href="../cn/Network.md">简体中文</a>
</p>

[← Contents](../../README.md)

# Refit

## Introduction

`Refit` is a REST-based HTTP client library. You can define routes on an interface and make HTTP requests by calling its methods, without writing the URL for each call. Java has OpenFeign in Spring Cloud; I had been looking for a similar C# library until I found `Refit`.

`GitHub`: [![GitHub Release](https://img.shields.io/github/v/release/reactiveui/refit)](https://github.com/reactiveui/refit)

`NuGet`: [![NuGet Version](https://img.shields.io/nuget/v/Refit)](https://www.nuget.org/packages/Refit/)

## Quick Start

### NuGet Packages

```c#
Refit.HttpClientFactory
Refit.Newtonsoft.Json
```

### Dependency Injection

```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

// Register ITestApi; create the interface first (see below)
builder.Services.AddRefitClient<ITestApi>(new RefitSettings())
				.ConfigureHttpClient(c =>
				{
					// Base address for requests (domain)
					c.BaseAddress = new Uri("http://127.0.0.1:9001");
					// Set the timeout
					c.Timeout = TimeSpan.FromSeconds(60);
				});
```


### Basic Usage

- Refit interface
```c#
public interface ITestApi
{
	/// <summary>
	/// Query
	/// </summary>
	/// <param name="id"></param>
	/// <returns></returns>
	[Get("/api/Test/Get")]
	Task<string> Get([Query] long id);

	/// <summary>
	/// Insert
	/// </summary>
	/// <param name="dto"></param>
	/// <returns></returns>
	[Post("/api/Test/Add")]
	Task<bool> Add([Body] DtoModel dto);
}
```
Define the request method, parameters, and route on the interface.

- Call the interface
```c#
public class TastService
{
	private readonly ITestApi _testApi;

	public TastService(ITestApi testApi)
	{
		_testApi = testApi;
	}

	/// <summary>
	/// Query
	/// </summary>
	/// <returns></returns>
	public async Task<string> Get()
	{
		// Business logic...

		// Make a GET request: http://127.0.0.1:9001/api/Test/Get?id=111
		return await _testApi.Get(111);
	}

	/// <summary>
	/// Add
	/// </summary>
	/// <returns></returns>
	public async Task<bool> Add()
	{
		DtoModel dto = new();
		// Business logic...

		// Make a POST request: http://127.0.0.1:9001/api/Test/Add
		return await _testApi.Add(dto);
	}
}
```
Inject `ITestApi` through the constructor and call its methods where needed. With `Refit`, you can define an interface and call its methods to make requests instead of writing many request URLs and conversions.
