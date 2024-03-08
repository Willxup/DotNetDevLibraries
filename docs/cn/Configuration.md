
🚧**装修中...**

# 目录

- [基础配置](#基础配置)
- [swagger文档配置](#swagger文档配置)
- [响应压缩](#响应压缩)
- [请求响应日志](#请求响应日志)

# 基础配置

- 配置文件(`appsettings.json`)
```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

builder.Host.ConfigureAppConfiguration((hostingContext, config) =>
{
    //设置appsetting.json
    config.SetBasePath(Directory.GetCurrentDirectory())
              .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
              .AddJsonFile($"appsettings.{EnvironmentHelper.GetEnv()}.json", optional: true, reloadOnChange: true)
              .AddEnvironmentVariables();
});
```

- Json配置
```c#
builder.Services.AddControllers()
	//取消默认驼峰
	.AddJsonOptions(options => { options.JsonSerializerOptions.PropertyNamingPolicy = null; })
	//设置默认Json解析为Newtonsoft.Json
	.AddNewtonsoftJson(options => 
	{
		options.SerializerSettings.ContractResolver = new DefaultContractResolver();
		options.SerializerSettings.DateFormatString = "yyyy-MM-dd HH:mm:ss.fff";
	});
```

- 健康检测
```c#
//添加健康检查
builder.Services.AddHealthChecks();

WebApplication app = builder.Build();

//设置健康检查路由
app.MapHealthChecks("/api/healthcheck");
```

# swagger文档配置

- `nuget`包
```c#
Swashbuckle.AspNetCore
Swashbuckle.AspNetCore.Newtonsoft
```
- 设置`xml`文档生成
```xml
<PropertyGroup>
	<TargetFramework>net6.0</TargetFramework>
	<GenerateDocumentationFile>True</GenerateDocumentationFile>
	<DocumentationFile>swagger.xml</DocumentationFile>
</PropertyGroup>
```

- `swagger`文档配置
```c#
/// <summary>
/// Swagger配置
/// </summary>
public static class SwaggerConfigExtension
{
	/// <summary>
	/// 配置Swagger，在program文件中调用
	/// </summary>
	/// <param name="services"></param>
	/// <param name="name">xml文档名称</param>
	/// <param name="basePath">xml文档位置</param>
	/// <returns></returns>
	public static IServiceCollection AddSwagger(this IServiceCollection services, string name, string basePath)
	{
		services.AddSwaggerGen(c =>
		{
			typeof(ApiVersions).GetEnumNames().ToList().ForEach(version =>
			{
				c.SwaggerDoc(version, new OpenApiInfo
				{
					// {ApiName} 定义成全局变量
					Title = $"{name} 接口文档",
					Description = $"{name} " + version,
					//TermsOfService = "None",
					Contact = new OpenApiContact { Name = name, Email = "test@willxup.top" }
				});
				// 按相对路径排序
				//c.OrderActionsBy(o => o.RelativePath);
			});
			//解决相同类名会报错的问题
			c.CustomSchemaIds(type => type.FullName);
			//配置的xml文件名
			var xmlPath = Path.Combine(basePath, $"{name}.xml");
			c.IncludeXmlComments(xmlPath, true);

			#region Token绑定到ConfigureServices
			//添加header验证信息
			//c.OperationFilter<SwaggerHeader>();

			var issuerName = "DotNetDevLibraries";
			var security = new OpenApiSecurityRequirement()
			{
				{
					new OpenApiSecurityScheme{
						Reference = new OpenApiReference {
									Type = ReferenceType.SecurityScheme,
									Id = issuerName},
						//Name = issuerName
					}, Array.Empty<string>()
				}
			};
			c.AddSecurityRequirement(security);

			//swagger文档token
			c.AddSecurityDefinition(issuerName, new OpenApiSecurityScheme
			{
				Description = "请在下框中输入Bearer {token}",
				Name = "Authorization", //jwt默认的参数名称
				In = ParameterLocation.Header, //jwt默认存放Authorization信息的位置(请求头中)
				Type = SecuritySchemeType.ApiKey,
				BearerFormat = "JWT",
				Scheme = "Bearer"
			});
			#endregion
		});

		services.AddSwaggerGenNewtonsoftSupport(); //使swagger支持newtonsoft.json

		return services;
	}
}
```

- `program.cs`配置
```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwagger("swagger", AppContext.BaseDirectory); //调用上面自定义的swagger配置，swagger是xml文档名称

WebApplication app = builder.Build();

app.UseSwagger();
app.UseSwaggerUI(option => 
{
	//根据版本名称正序 遍历展示
	options.SwaggerEndpoint($"/swagger/swagger.json", $"swagger");

	// 将swagger首页,设置成自定义的页面: index.html               
	// 路径配置，设置为空，表示直接在根域名（localhost:8000）访问该文件,注意localhost:8000/swagger是访问不到的，去launchSettings.json把launchUrl去掉
	options.RoutePrefix = "";

	// Display
	options.DefaultModelExpandDepth(2);
	options.DefaultModelRendering(ModelRendering.Model);
	options.DefaultModelsExpandDepth(-1);
	options.DisplayOperationId();
	options.DisplayRequestDuration();
	options.DocExpansion(DocExpansion.None);
	options.ShowExtensions();
});

```

# 响应压缩

```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

//配置压缩方式
services.Configure<BrotliCompressionProviderOptions>(options =>
{
	options.Level = CompressionLevel.Fastest; //压缩应该尽快完成，即使生成的输出未以最佳方式压缩。
});

//配置压缩方式
services.Configure<GzipCompressionProviderOptions>(options =>
{
	options.Level = CompressionLevel.Fastest; //压缩应该尽快完成，即使生成的输出未以最佳方式压缩。
});

//响应压缩
services.AddResponseCompression(options =>
{
	options.Providers.Add<BrotliCompressionProvider>(); //br压缩
	options.Providers.Add<GzipCompressionProvider>(); //gzip压缩
});

WebApplication app = builder.Build();

//使用响应压缩
app.UseResponseCompression();

```

# 请求响应日志

- 配置请求响应日志的中间件
```c#
/// <summary>
/// 请求和响应
/// </summary>
public class RequestLogMiddleware
{
	/// <summary>
	/// 委托
	/// </summary>
	private readonly RequestDelegate _next;
	/// <summary>
	/// 请求白名单TokenPass
	/// </summary>
	private readonly string[] REQUEST_FILTER_WHITE_URL;

	/// <summary>
	/// 构造
	/// </summary>
	/// <param name="next"></param>
	/// <param name="configuration"></param>
	public RequestLogMiddleware(RequestDelegate next, IConfiguration configuration)
	{
		_next = next;
		REQUEST_FILTER_WHITE_URL = configuration.GetValue<string>("RequestFilter.WhiteList").ToLower().Split(","); //从配置中获取白名单
	}

	/// <summary>
	/// 通道中间件
	/// </summary>
	/// <param name="context"></param>
	/// <returns></returns>
	public async Task InvokeAsync(HttpContext context)
	{
		//健康检查跳过
		if (context.Request.Path.Value.ToLower().Contains("/api/healthcheck"))
		{
			await _next(context);
			return;
		}

		//白名单跳过
		if (REQUEST_FILTER_WHITE_URL.Contains(context.Request.Path.Value.ToLower()))
		{
			await _next(context);
			return;
		}

		// 拦截API接口
		if (context.Request.Path.Value.ToLower().Contains("/api"))
		{
			context.Request.EnableBuffering();

			string connectId = context.Connection.Id;

			using Stream originalBody = context.Response.Body;
			try
			{
				// 请求
				await RecordRequestLog(context.Request, connectId);

				using var ms = new MemoryStream();
				context.Response.Body = ms;

				// 访问下一中间件
				await _next(context);

				try
				{
					// 响应
					await RecordResponseLog(context.Response, ms, connectId);

					ms.Position = 0;
					await ms.CopyToAsync(originalBody);
				}
				catch (Exception ex)
				{
					//日志
					Console.WriteLine(ex.Message);
				}
			}
			catch (Exception ex)
			{
				//日志
				Console.WriteLine(ex.Message);
				await _next(context);
			}
			finally
			{
				context.Response.Body = originalBody;
			}
		}
		else
		{
			await _next(context);
			return;
		}
	}

	/// <summary>
	/// 请求日志
	/// </summary>
	/// <param name="request"></param>
	/// <param name="connectId"></param>
	/// <returns></returns>
	private static async Task RecordRequestLog(HttpRequest request, string connectId)
	{
		try
		{
			//log内容
			string content = string.Empty;

			if (request.Path.Value.ToLower().Contains("excel"))
			{
				content = $"ConnectId:{connectId}\r\n" +
						  $"QueryData:{request.Path + request.QueryString}\r\n" +
						  $"RequestPath:{request.GetDisplayUrl()}\r\n";
			}
			else
			{

				//读取请求体
				var sr = new StreamReader(request.Body);

				content = $"ConnectId:{connectId}\r\n" +
						  $"QueryData:{request.Path + request.QueryString}\r\n" +
						  $"BodyData:{await sr.ReadToEndAsync()}\r\n" +
						  $"RequestPath:{request.GetDisplayUrl()}\r\n";
			}



			if (!string.IsNullOrEmpty(content))
			{
				//写入控制台，可修改为写入日志
				ConsoleLog("Request Data", content);


				//将请求体的流重置
				request.Body.Position = 0;
			}
		}
		catch (Exception) { }
	}

	/// <summary>
	/// 响应日志
	/// </summary>
	/// <param name="response"></param>
	/// <param name="ms"></param>
	/// <param name="connectId"></param>
	/// <returns></returns>
	private static async Task RecordResponseLog(HttpResponse response, MemoryStream ms, string connectId)
	{
		try
		{
			//设置流的位置
			ms.Position = 0;

			//读取响应
			string body = await new StreamReader(ms).ReadToEndAsync();

			if (!string.IsNullOrEmpty(body))
			{
				string content = $"ConnectId:{connectId}\r\n" +
								 $"BodyData:{body}\r\n";

				//写入控制台，可修改为写入日志
				ConsoleLog("Request Data", content);
			}

		}
		catch (Exception) { }
	}

	/// <summary>
	/// 控制台日志
	/// </summary>
	/// <param name="type"></param>
	/// <param name="content"></param>
	private static void ConsoleLog(string type, string content)
	{
		Console.ForegroundColor = ConsoleColor.Green;
		Console.Write(DateTime.Now.ToFormattedString());
		Console.ForegroundColor = ConsoleColor.Red;
		Console.Write($"【{type}】：\n");
		Console.ResetColor();
		Console.WriteLine(content);
	}
}
```

- `program.cs`配置
```c#
WebApplication app = builder.Build();

//使用 请求响应日志 中间件
app.UseMiddleware<RequestLogMiddleware>();
```

