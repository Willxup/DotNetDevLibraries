
🚧**装修中...**

# SqlSugar

## 介绍

`SqlSugar` 是一款 老牌` .NET`开源`ORM`框架，支持非常多的数据库，一些国产数据库也得到了很好的支持。`SqlSugar`使用起来也是非常的简单，会使用`Linq`基本可以快速上手。

另外，`SqlSugar`解决问题的速度也很快，几乎当天解决，经过这么多年的迭代，几乎可以非常稳定的运行。

`Github`：[![GitHub Release](https://img.shields.io/github/v/release/DotNetNext/SqlSugar)](https://github.com/DotNetNext/SqlSugar)

`Nuget`：[![NuGet Version](https://img.shields.io/nuget/v/SqlSugarCore)](https://www.nuget.org/packages/SqlSugarCore/)

文档：[https://www.donet5.com/Home/Doc](https://www.donet5.com/Home/Doc)

## 快速上手

### IOC注入

```c#
//builder
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

builder.Services.AddSingleton<ISqlSugarClient>(sugar =>
{
	//开启WhereIF功能
	StaticConfig.EnableAllWhereIF = true;

	//SqlSugarScope 单例模式
	return new SqlSugarScope(new ConnectionConfig
	{
    ConnectionString = connectionString,
    DbType = DbType.MySql, //数据库类型
    IsAutoCloseConnection = true, //自动释放
    LanguageType = LanguageType.English //报错提示
	}, 
	db => 
	{
		//设置额外配置
		db.CurrentConnectionConfig.MoreSettings = new ConnMoreSettings() 
		{ 
			IsAutoRemoveDataCache = true //自动移除缓存
		};

		//扩展
		db.CurrentConnectionConfig.ConfigureExternalServices = new ConfigureExternalServices()
		{
			SqlFuncServices = DbExtension.CustomSqlFunc()
		};
		
		//AOP 打印SQL在控制台
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

### 简单使用

- 实体类及DTO
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

- 业务逻辑
```c#
public interface ITestService { }

public class TestService : ITestService
{
	private SqlSugarScope _dbContext;

	public TestService(ISqlSugarClient dbContext)
	{
		_dbContext = dbContext as SqlSugarScope; //接口转为Scope
	}

	/// <summary>
	/// 获取
	/// </summary>
	/// <returns></returns>
	public async Task<Table_Name> Get()
	{
		return await _dbContext.Queryable<Table_Name>()
			.Where(t => t.Name = "test")
			.ToListAsync();
	}

	/// <summary>
	/// 获取DTO
	/// </summary>
	/// <returns></returns>
	public async Task<Table_Name> GetDTO()
	{
		return await _dbContext.Queryable<Table_Name>()
			.Where(t => t.Name = "test")
			.Select(t => new TableDTO
			{
				Id = t.Id,
				Name = t.Name
			})
			.ToListAsync();
	}

	/// <summary>
	/// 新增
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
	/// 编辑
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
	/// 删除
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

## 封装扩展

为了快速高效的开发，我为`SqlSugar`写了一套封装扩展`nuget`包，可以更方便的使用。这套基于[`SqlSugar`](https://github.com/DotNetNext/SqlSugar)和`C#`特性 配置`dto`模型，只需要在模型上配置表信息，即可自动拼接SQL，实现查询插入更新等功能，实现高效项目开发和迭代。

`Github`：[![GitHub Release](https://img.shields.io/github/v/release/Willxup/SqlSugar.Attributes)](https://github.com/Willxup/SqlSugar.Attributes)

`Nuget`：[![NuGet Version](https://img.shields.io/nuget/v/SqlSugar.Attributes)](https://www.nuget.org/packages/SqlSugar.Attributes)


### 简单实用

使用`SqlSugar.Attributes`可以直接配置DTO模型，配置完成后，将会自动映射查询条件及结果。这里仅简单展示查询方式，更多的细节可以前往[SqlSugar.Attributes](https://github.com/Willxup/SqlSugar.Attributes)查看。

- 引入Nuget包
```c#
SqlSugar.Attributes
```

- 创建查询的查询条件类

```c#
[DbDefaultOrderBy("t_Name", DbSortWay.DESC)]
//[DbDefaultOrderBy("t.t_Name", DbSortWay.DESC)]
public class Search
{
	//[DbTableAlias("t")] //表别名，多表关联使用，单表需要查询时指定表别名
	[DbQueryField("t_Name")] //字段名
	[DbQueryOperator(DbOperator.Like)]  //操作符, Like查询
	public string Name { get; set; }
	//[DbTableAlias("t")] //表别名，多表关联使用，单表需要查询时指定表别名
	[DbQueryField("t_CodeId")] //字段名
	[DbQueryOperator(DbOperator.In)] //操作符，IN查询
	public List<long> CodeIds { get; set; }
}
```

- 创建查询的结果类

```c#
public class Result
{
	//[DbTableAlias("t")] //表别名，多表关联使用，单表需要查询时指定表别名
	[DbQueryField("t_Id")] //字段名
	public long Id { get; set; }
	//[DbTableAlias("t")] //表别名，多表关联使用，单表需要查询时指定表别名
	[DbQueryField("t_Name")] //字段名
	public string Name { get; set; }
    //子查询，如果指定了表别名，可以使用表别名，否则直接写表名
	//[DbSubQuery("(SELECT name FROM Table_Name2 WHERE t.t_Id = Id)")]
    [DbSubQuery("(SELECT name FROM Table_Name2 WHERE Table_Name.t_Id = Id)")]
    public string OtherName { get; set; }
}
```

- 查询方法

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
            .Where(search) //查询条件
            .OrderBy(search) //排序
            .Select(new Result()) //查询结果
            .ToListAsync();
    }
}
```

注：在使用`SqlSugar.Attribute`时，依然可以使用`SqlSugar`的`Where`，`GroupBy`，`OrderBy`等方法。

- 正常查询SQL

```sql
SELECT 
t_Id as Id, 
t_Name as Name, 
(SELECT name FROM Table_Name2 WHERE Table_Name.t_Id = Id) AS OtherName
FROM Table_Name
WHERE t_Name LIKE '%@Name%' AND t_CodeId IN (@CodeIds)
ORDER BY t_Name DESC;
```

- 指定表别名查询SQL

```sql
SELECT 
t.t_Id as Id, 
t.t_Name as Name,
(SELECT name FROM Table_Name2 WHERE t.t_Id = Id) AS OtherName
FROM Table_Name t
WHERE t.t_Name LIKE '%@Name%' AND t.t_CodeId IN (@CodeIds)
ORDER BY t.t_Name DESC;
```
