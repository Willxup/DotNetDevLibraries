<p align="center">
  <a href="../en/Configuration.md"><strong>English</strong></a> ｜ <a href="../cn/Configuration.md">简体中文</a>
</p>

[← Contents](../../README.md)

# Contents

- [Basic Configuration](#basic-configuration)
- [Swagger Configuration](#swagger-configuration)
- [Response Compression](#response-compression)
- [Request and Response Logging](#request-and-response-logging)

# Basic Configuration

- Configuration file (`appsettings.json`)
```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

builder.Host.ConfigureAppConfiguration((hostingContext, config) =>
{
    // Configure appsetting.json
    config.SetBasePath(Directory.GetCurrentDirectory())
              .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
              .AddJsonFile($"appsettings.{EnvironmentHelper.GetEnv()}.json", optional: true, reloadOnChange: true)
              .AddEnvironmentVariables();
});
```

- JSON configuration
```c#
builder.Services.AddControllers()
	// Disable the default camel case naming
	.AddJsonOptions(options => { options.JsonSerializerOptions.PropertyNamingPolicy = null; })
	// Use Newtonsoft.Json as the default JSON parser
	.AddNewtonsoftJson(options =>
	{
		options.SerializerSettings.ContractResolver = new DefaultContractResolver();
		options.SerializerSettings.DateFormatString = "yyyy-MM-dd HH:mm:ss.fff";
	});
```

- Health checks
```c#
// Add health checks
builder.Services.AddHealthChecks();

WebApplication app = builder.Build();

// Set the health check route
app.MapHealthChecks("/api/healthcheck");
```

# Swagger Configuration

- `nuget` packages
```c#
Swashbuckle.AspNetCore
Swashbuckle.AspNetCore.Newtonsoft
```
- Enable `xml` documentation generation
```xml
<PropertyGroup>
	<TargetFramework>net6.0</TargetFramework>
	<GenerateDocumentationFile>True</GenerateDocumentationFile>
	<DocumentationFile>swagger.xml</DocumentationFile>
</PropertyGroup>
```

- `swagger` configuration
```c#
/// <summary>
/// Swagger configuration
/// </summary>
public static class SwaggerConfigExtension
{
	/// <summary>
	/// Configure Swagger; call this from the program file
	/// </summary>
	/// <param name="services"></param>
	/// <param name="name">XML document name</param>
	/// <param name="basePath">XML document location</param>
	/// <returns></returns>
	public static IServiceCollection AddSwagger(this IServiceCollection services, string name, string basePath)
	{
		services.AddSwaggerGen(c =>
		{
			typeof(ApiVersions).GetEnumNames().ToList().ForEach(version =>
			{
				c.SwaggerDoc(version, new OpenApiInfo
				{
					// Define {ApiName} as a global variable
					Title = $"{name} 接口文档", // {name} API documentation.
					Description = $"{name} " + version,
					//TermsOfService = "None",
					Contact = new OpenApiContact { Name = name, Email = "test@willxup.top" }
				});
				// Sort by relative path
				//c.OrderActionsBy(o => o.RelativePath);
			});
			// Avoid errors when classes share the same name
			c.CustomSchemaIds(type => type.FullName);
			// Configured XML file name
			var xmlPath = Path.Combine(basePath, $"{name}.xml");
			c.IncludeXmlComments(xmlPath, true);

			#region Token binding in ConfigureServices
			// Add header authentication information
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

			// Swagger documentation token
			c.AddSecurityDefinition(issuerName, new OpenApiSecurityScheme
			{
				Description = "请在下框中输入Bearer {token}", // Enter Bearer {token} in the box below.
				Name = "Authorization", // Default JWT parameter name
				In = ParameterLocation.Header, // Default location for JWT Authorization information (request header)
				Type = SecuritySchemeType.ApiKey,
				BearerFormat = "JWT",
				Scheme = "Bearer"
			});
			#endregion
		});

		services.AddSwaggerGenNewtonsoftSupport(); // Enable Newtonsoft.Json support in Swagger

		return services;
	}
}
```

- `program.cs` configuration
```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwagger("swagger", AppContext.BaseDirectory); // Call the custom Swagger configuration above; swagger is the XML document name

WebApplication app = builder.Build();

app.UseSwagger();
app.UseSwaggerUI(option =>
{
	// Display versions in ascending order by name
	options.SwaggerEndpoint($"/swagger/swagger.json", $"swagger");

	// Set the Swagger home page to a custom page: index.html
	// An empty route prefix serves it at the root (localhost:8000), not localhost:8000/swagger. Remove launchUrl from launchSettings.json.
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

# Response Compression

```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

// Configure compression
services.Configure<BrotliCompressionProviderOptions>(options =>
{
	options.Level = CompressionLevel.Fastest; // Compress as quickly as possible, even if the output is not compressed as much as it could be.
});

// Configure compression
services.Configure<GzipCompressionProviderOptions>(options =>
{
	options.Level = CompressionLevel.Fastest; // Compress as quickly as possible, even if the output is not compressed as much as it could be.
});

// Response compression
services.AddResponseCompression(options =>
{
	options.Providers.Add<BrotliCompressionProvider>(); // Brotli compression
	options.Providers.Add<GzipCompressionProvider>(); // Gzip compression
});

WebApplication app = builder.Build();

// Use response compression
app.UseResponseCompression();

```

# Request and Response Logging

- Configure request and response logging middleware
```c#
/// <summary>
/// Request and response
/// </summary>
public class RequestLogMiddleware
{
	/// <summary>
	/// Delegate
	/// </summary>
	private readonly RequestDelegate _next;
	/// <summary>
	/// Request allowlist TokenPass
	/// </summary>
	private readonly string[] REQUEST_FILTER_WHITE_URL;

	/// <summary>
	/// Constructor
	/// </summary>
	/// <param name="next"></param>
	/// <param name="configuration"></param>
	public RequestLogMiddleware(RequestDelegate next, IConfiguration configuration)
	{
		_next = next;
		REQUEST_FILTER_WHITE_URL = configuration.GetValue<string>("RequestFilter.WhiteList").ToLower().Split(","); // Get the allowlist from configuration
	}

	/// <summary>
	/// Pipeline middleware
	/// </summary>
	/// <param name="context"></param>
	/// <returns></returns>
	public async Task InvokeAsync(HttpContext context)
	{
		// Skip health checks
		if (context.Request.Path.Value.ToLower().Contains("/api/healthcheck"))
		{
			await _next(context);
			return;
		}

		// Skip allowlisted requests
		if (REQUEST_FILTER_WHITE_URL.Contains(context.Request.Path.Value.ToLower()))
		{
			await _next(context);
			return;
		}

		// Intercept API requests
		if (context.Request.Path.Value.ToLower().Contains("/api"))
		{
			context.Request.EnableBuffering();

			string connectId = context.Connection.Id;

			using Stream originalBody = context.Response.Body;
			try
			{
				// Request
				await RecordRequestLog(context.Request, connectId);

				using var ms = new MemoryStream();
				context.Response.Body = ms;

				// Call the next middleware
				await _next(context);

				try
				{
					// Response
					await RecordResponseLog(context.Response, ms, connectId);

					ms.Position = 0;
					await ms.CopyToAsync(originalBody);
				}
				catch (Exception ex)
				{
					// Log
					Console.WriteLine(ex.Message);
				}
			}
			catch (Exception ex)
			{
				// Log
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
	/// Request log
	/// </summary>
	/// <param name="request"></param>
	/// <param name="connectId"></param>
	/// <returns></returns>
	private static async Task RecordRequestLog(HttpRequest request, string connectId)
	{
		try
		{
			// Log content
			string content = string.Empty;

			if (request.Path.Value.ToLower().Contains("excel"))
			{
				content = $"ConnectId:{connectId}\r\n" +
						  $"QueryData:{request.Path + request.QueryString}\r\n" +
						  $"RequestPath:{request.GetDisplayUrl()}\r\n";
			}
			else
			{

				// Read the request body
				var sr = new StreamReader(request.Body);

				content = $"ConnectId:{connectId}\r\n" +
						  $"QueryData:{request.Path + request.QueryString}\r\n" +
						  $"BodyData:{await sr.ReadToEndAsync()}\r\n" +
						  $"RequestPath:{request.GetDisplayUrl()}\r\n";
			}



			if (!string.IsNullOrEmpty(content))
			{
				// Write to the console; change this to write to a log if needed
				ConsoleLog("Request Data", content);


				// Reset the request body stream
				request.Body.Position = 0;
			}
		}
		catch (Exception) { }
	}

	/// <summary>
	/// Response log
	/// </summary>
	/// <param name="response"></param>
	/// <param name="ms"></param>
	/// <param name="connectId"></param>
	/// <returns></returns>
	private static async Task RecordResponseLog(HttpResponse response, MemoryStream ms, string connectId)
	{
		try
		{
			// Set the stream position
			ms.Position = 0;

			// Read the response
			string body = await new StreamReader(ms).ReadToEndAsync();

			if (!string.IsNullOrEmpty(body))
			{
				string content = $"ConnectId:{connectId}\r\n" +
								 $"BodyData:{body}\r\n";

				// Write to the console; change this to write to a log if needed
				ConsoleLog("Request Data", content);
			}

		}
		catch (Exception) { }
	}

	/// <summary>
	/// Console log
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

- `program.cs` configuration
```c#
WebApplication app = builder.Build();

// Use the request and response logging middleware
app.UseMiddleware<RequestLogMiddleware>();
```

