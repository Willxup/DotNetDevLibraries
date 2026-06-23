<p align="center">
  <a href="../en/DbORM.md"><strong>English</strong></a> ｜ <a href="../cn/DbORM.md">简体中文</a>
</p>

[← Contents](../../README.md)

# SqlSugar

## Introduction

`SqlSugar` is a long-standing open-source ORM for .NET. It supports many databases, including some developed in China. If you know LINQ, it is easy to get started.

The author notes that issues are often resolved on the same day and that years of updates have made `SqlSugar` stable.

Supported databases include `Oracle`, `MySQL`, `SQL Server`, `PostgreSQL`, `SQLite`, `Dameng`, and `Kingbase`.

`GitHub`: [![GitHub Release](https://img.shields.io/github/v/release/DotNetNext/SqlSugar)](https://github.com/DotNetNext/SqlSugar)

`NuGet`: [![NuGet Version](https://img.shields.io/nuget/v/SqlSugarCore)](https://www.nuget.org/packages/SqlSugarCore/)

Documentation: [https://www.donet5.com/Home/Doc](https://www.donet5.com/Home/Doc)

## Quick Start (MySQL Example)

### NuGet Packages

```c#
SqlSugarCoreNoDrive
MySqlConnector // MySQL example
```

### Dependency Injection

```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

builder.Services.AddSingleton<ISqlSugarClient>(sugar =>
{
	// Enable WhereIF
	StaticConfig.EnableAllWhereIF = true;

	// Use SqlSugarScope as a singleton
	return new SqlSugarScope(new ConnectionConfig
	{
    ConnectionString = connectionString,
    DbType = DbType.MySql, // Database type
    IsAutoCloseConnection = true, // Release automatically
    LanguageType = LanguageType.English // Error messages
	},
	db =>
	{
		// Set additional configuration
		db.CurrentConnectionConfig.MoreSettings = new ConnMoreSettings()
		{
			IsAutoRemoveDataCache = true // Remove the cache automatically
		};

		// Extensions
		db.CurrentConnectionConfig.ConfigureExternalServices = new ConfigureExternalServices()
		{
			SqlFuncServices = DbExtension.CustomSqlFunc()
		};

		// AOP: Print SQL to the console
		db.Aop.OnLogExecuting = (sql, parameters) =>
		{
			Console.ForegroundColor = ConsoleColor.Green;
			Console.Write(DateTime.Now.ToFormattedString());
			Console.ForegroundColor = ConsoleColor.Red;
			Console.Write("【SQL】：");
			Console.ResetColor();
			Console.WriteLine(sql);
			if (parameters != null && parameters?.Length > 0)
			{
				Console.ForegroundColor = ConsoleColor.Green;
				Console.Write(DateTime.Now.ToFormattedString());
				Console.ForegroundColor = ConsoleColor.Red;
				Console.Write("【Paramters】：");
				Console.ResetColor();
				Console.WriteLine(string.Join(",", parameters?.Select(it => "【" + it.ParameterName + "=" + it.Value + "】")));
			}
		};
	});
});
```

### Basic Usage

- Entity class and DTO
```c#
public class Table_Name
{
	[SugarColumn(IsPrimaryKey=true,IsIdentity=true)]
	public long Id { get; set; }
	public string Name { get; set; }
	public string Address { get; set; }
	public byte IsDelete { get; set; }
}

public class TableDTO
{
	public long Id { get; set; }
	public string Name { get; set; }
}
```

- Business logic
```c#
public interface ITestService { }

public class TestService : ITestService
{
	private SqlSugarScope _dbContext;

	public TestService(ISqlSugarClient dbContext)
	{
		_dbContext = dbContext as SqlSugarScope; // Convert the interface to Scope
	}

	/// <summary>
	/// Get
	/// </summary>
	/// <returns></returns>
	public async Task<List<Table_Name>> Get()
	{
		return await _dbContext.Queryable<Table_Name>()
			.Where(t => t.Name == "test")
			.ToListAsync();
	}

	/// <summary>
	/// Get a DTO
	/// </summary>
	/// <returns></returns>
	public async Task<List<TableDTO>> GetDTO()
	{
		return await _dbContext.Queryable<Table_Name>()
			.Where(t => t.Name == "test")
			.Select(t => new TableDTO
			{
				Id = t.Id,
				Name = t.Name
			})
			.ToListAsync();
	}

	/// <summary>
	/// Add
	/// </summary>
	/// <returns></returns>
	public async Task<int> Add(string name, string address)
	{
		Table_Name entity = new()
		{
			Name = name,
			Address = address,
			IsDelete = 0
		};
		return await _dbContext.Insertable(entity).ExecuteCommandAsync();
	}

	/// <summary>
	/// Edit
	/// </summary>
	/// <returns></returns>
	public async Task<bool> Edit(long id, string name, string address)
	{
		return await _dbContext.Updateable<Table_Name>()
			.SetColumns(t => t.Name == name)
			.SetColumns(t => t.Address == address)
			.Where(t => t.Id == id)
			.ExecuteCommandHasChangeAsync();
	}

	/// <summary>
	/// Delete
	/// </summary>
	/// <returns></returns>
	public async Task<bool> Delete(long id)
	{
		return await _dbContext.Deleteable<Table_Name>()
			.Where(t => t.Id == id)
			.ExecuteCommandHasChangeAsync();
	}
}
```

## Helper Extensions

I created a `NuGet` package with helper extensions to make `SqlSugar` easier to use. It uses [`SqlSugar`](https://github.com/DotNetNext/SqlSugar) and `C#` attributes to configure DTO models. Add table information to a model, and it builds SQL for queries, inserts, and updates to support faster project development and iteration.

`GitHub`: [![GitHub Release](https://img.shields.io/github/v/release/Willxup/SqlSugar.Attributes)](https://github.com/Willxup/SqlSugar.Attributes)

`NuGet`: [![NuGet Version](https://img.shields.io/nuget/v/SqlSugar.Attributes)](https://www.nuget.org/packages/SqlSugar.Attributes)


### Basic Usage

Use `SqlSugar.Attributes` to configure DTO models directly. Once configured, it maps query conditions and results automatically. This example only shows a basic query. For more details, see [SqlSugar.Attributes](https://github.com/Willxup/SqlSugar.Attributes) or the [FastAdminAPI](https://github.com/Willxup/FastAdminAPI) project.

- Add the NuGet package
```c#
SqlSugar.Attributes
```

- Create a query conditions class

```c#
[DbDefaultOrderBy("t_Name", DbSortWay.DESC)]
//[DbDefaultOrderBy("t.t_Name", DbSortWay.DESC)]
public class Search
{
	//[DbTableAlias("t")] // Table alias for joins; for one table, specify the alias when querying
	[DbQueryField("t_Name")] // Field name
	[DbQueryOperator(DbOperator.Like)]  // Operator: Like query
	public string Name { get; set; }
	//[DbTableAlias("t")] // Table alias for joins; for one table, specify the alias when querying
	[DbQueryField("t_CodeId")] // Field name
	[DbQueryOperator(DbOperator.In)] // Operator: IN query
	public List<long> CodeIds { get; set; }
}
```

- Create a query result class

```c#
public class Result
{
	//[DbTableAlias("t")] // Table alias for joins; for one table, specify the alias when querying
	[DbQueryField("t_Id")] // Field name
	public long Id { get; set; }
	//[DbTableAlias("t")] // Table alias for joins; for one table, specify the alias when querying
	[DbQueryField("t_Name")] // Field name
	public string Name { get; set; }
    // Subquery: use the table alias if specified, otherwise use the table name
	//[DbSubQuery("(SELECT name FROM Table_Name2 WHERE t.t_Id = Id)")]
    [DbSubQuery("(SELECT name FROM Table_Name2 WHERE Table_Name.t_Id = Id)")]
    public string OtherName { get; set; }
}
```

- Query method

```c#
public class Test
{
    /// <summary>
    /// SqlSugar dbcontext
    /// </summary>
	private readonly ISqlSugarClient _dbContext;

    public async Task<List<Result>> Get(Search search)
    {
        return await _dbContext.Queryable<Table_Name>()
            .Where(search) // Query conditions
            .OrderBy(search) // Sort order
            .Select(new Result()) // Query results
            .ToListAsync();
    }
}
```

Note: When using `SqlSugar.Attributes`, you can still use `SqlSugar` methods such as `Where`, `GroupBy`, and `OrderBy`.

- SQL query

```sql
SELECT
t_Id as Id,
t_Name as Name,
(SELECT name FROM Table_Name2 WHERE Table_Name.t_Id = Id) AS OtherName
FROM Table_Name
WHERE t_Name LIKE '%@Name%' AND t_CodeId IN (@CodeIds)
ORDER BY t_Name DESC;
```

- SQL query with a table alias

```sql
SELECT
t.t_Id as Id,
t.t_Name as Name,
(SELECT name FROM Table_Name2 WHERE t.t_Id = Id) AS OtherName
FROM Table_Name t
WHERE t.t_Name LIKE '%@Name%' AND t.t_CodeId IN (@CodeIds)
ORDER BY t.t_Name DESC;
```
