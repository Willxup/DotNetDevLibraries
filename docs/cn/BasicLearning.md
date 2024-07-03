# 目录

- [官网网站](#官网网站)
- [开发小技巧](#开发小技巧)
  - [LINQ](#LINQ)
  - [语法糖](#语法糖)


# 官网网站

官方文档是最正确的文档，虽然微软的机翻实在不怎么行，不过还是建议尽量看官方文档，解决不了的再去查其他资料。

- [c#官网文档](https://learn.microsoft.com/zh-cn/dotnet/csharp/)
- [c#官方在线代码运行](https://learn.microsoft.com/zh-cn/dotnet/csharp/tour-of-csharp/#code-try-0)
- [.net官方文档](https://learn.microsoft.com/zh-cn/dotnet/fundamentals/)
- [.net官方下载](https://dotnet.microsoft.com/zh-cn/download/dotnet)

# 开发小技巧

这里总结了很多开发中能实际用到的一些小技巧，可以大大简化代码量，提升效率。

## LINQ

> C#语言中的LINQ（Language-Integrated Query）是一种强大的查询语言，它提供了一种统一的编程模型，使得数据查询变得更加直观和简洁。

官方文档：[https://learn.microsoft.com/zh-cn/dotnet/csharp/linq/](https://learn.microsoft.com/zh-cn/dotnet/csharp/linq/)

### FROM

> 任何数据源，包括对象集合、数据库、XML等。

### WHERE

> 条件查询

```cs
// 示例数据源  
List<Student> students = new List<Student>  
{  
    new Student { Name = "Alice", Age = 25, Grade = "A" },  
    new Student { Name = "Bob", Age = 30, Grade = "B" },  
    new Student { Name = "Barry", Age = 35, Grade = "C" },  
    new Student { Name = "Charlie", Age = 22, Grade = "A" },  
    new Student { Name = "David", Age = 21, Grade = "C" },  
    new Student { Name = "Damon", Age = 28, Grade = "B" },  
    new Student { Name = "Echo", Age = 18, Grade = "C" }  
};  

// 使用WHERE筛选出成绩为A的学生  
var result1 = students.Where(student => student.Grade = "A");  

// 使用WHERE筛选出年龄大于20的学生  
var result2 = students.Where(student => student.Age > 20);  

// 使用WHERE筛选出名字包含'D'的学生  
var result3 = students.Where(student => student.Name.Contains("D"));  

// 使用WHERE筛选出名字为'Bob'和'Echo'的学生  
List<string> nameList = new() { "Bob", "Echo" };  
var result4 = students.Where(student => nameList.Contains(student.Name));
```

### SELECT

> 结果查询

```cs
// 示例数据源  
List<Student> students = new List<Student>  
{  
    new Student { Name = "Alice", Age = 25, Grade = "A" },  
    new Student { Name = "Bob", Age = 30, Grade = "B" },  
    new Student { Name = "Barry", Age = 35, Grade = "C" },  
    new Student { Name = "Charlie", Age = 22, Grade = "A" },  
    new Student { Name = "David", Age = 21, Grade = "C" },  
    new Student { Name = "Damon", Age = 28, Grade = "B" },  
    new Student { Name = "Echo", Age = 18, Grade = "C" }  
};  

// 使用SELECT创建一个新的匿名类，并输出为集合，一般配合Where使用  
var result1 = students.Select(student =>   
    new   
    {  
        student.Name,  
        student.Age  
    });  

// 使用SELECT创建一个新的指定类，并输出为集合  
var result2 = students.Select(student => new StudentDto()  
    {  
        StudentName = student.Name,  
        StudentAge = student.Age  
    });
```

### GROUPBY

> 分组查询

```cs
// 示例数据源  
List<Student> students = new List<Student>  
{  
    new Student { Name = "Alice", Age = 25, Grade = "A" },  
    new Student { Name = "Bob", Age = 30, Grade = "B" },  
    new Student { Name = "Barry", Age = 35, Grade = "C" },  
    new Student { Name = "Charlie", Age = 22, Grade = "A" },  
    new Student { Name = "David", Age = 21, Grade = "C" },  
    new Student { Name = "Damon", Age = 28, Grade = "B" },  
    new Student { Name = "Echo", Age = 18, Grade = "C" }  
};  

// 使用GROUP BY按成绩进行分组查询  
var groupedByGrade = students.GroupBy(student => student.Grade);
```

### ORDERBY

> 排序

```cs
// 示例数据源  
List<Student> students = new List<Student>  
{  
    new Student { Name = "Alice", Age = 25, Grade = "A" },  
    new Student { Name = "Bob", Age = 30, Grade = "B" },  
    new Student { Name = "Barry", Age = 35, Grade = "C" },  
    new Student { Name = "Charlie", Age = 22, Grade = "A" },  
    new Student { Name = "David", Age = 21, Grade = "C" },  
    new Student { Name = "Damon", Age = 28, Grade = "B" },  
    new Student { Name = "Echo", Age = 18, Grade = "C" }  
};  

// 使用LINQ的OrderBy进行排序  
var result1 = students.OrderBy(student => student.Age);  

// 使用LINQ的OrderByDescending进行降序排序  
var result2 = students.OrderByDescending(student => student.Age);
```

### JOIN

> `Join`：用于执行内连接操作，它会返回两个数据源中满足连接条件的元素的交集
> 
> `GroupJoin`：用于执行左外连接（left outer join）操作，它会返回左边数据源的所有元素，以及每个元素所匹配的右边数据源的元素组成的集合。(嵌套)

```cs
// 示例数据源  
List<Department> departments = new List<Department>  
{  
    new Department { ID = 1, Name = "HR" },  
    new Department { ID = 2, Name = "IT" }  
};  
// 示例数据源  
List<Employee> employees = new List<Employee>  
{  
    new Employee { DepartmentID = 1, Name = "Alice" },  
    new Employee { DepartmentID = 2, Name = "Bob" },  
    new Employee { DepartmentID = 1, Name = "Charlie" },  
    new Employee { DepartmentID = 3, Name = "David" }  
};  

// 使用Join，将部门和员工相结合，获取部门名称和员工名称的集合  
var joinQuery = departments  
    .Join(employees,   
        department => department.ID,   
        employee => employee.DepartmentID,   
        (department, employee) => new { Department = department.Name, Employee = employee.Name }  
    );  

// 使用GroupJoin，将部门和员工相结合，返回所有的部门，并返回每个部门相关联的员工集合(嵌套)  
var groupJoinQuery = departments  
    .GroupJoin(employees,   
        department => department.ID,   
        employee => employee.DepartmentID,   
        (department, employeeGroup) => new   
            {   
                Department = department.Name,   
                Employees = employeeGroup.Select(e => e.Name).ToList()   
            }  
    );
```

### 结果转换

```cs
// ToList(): 将结果转换为List集合。  
List<Student> resultList = result.ToList();  

// ToDictionary(): 将结果转换为Dictionary字典。  
Dictionary<string, int> resultDictionary = students  
    .ToDictionary(student => student.Name, student => student.Age);  

// ToArray(): 将结果转换为数组。  
Student[] resultArray = result.ToArray();  

// First(): 获取结果中的第一个元素。  
Student firstStudent = result.First();  

// FirstOrDefault(): 获取结果中的第一个元素，如果结果为空则返回默认值。  
Student firstStudent = result.FirstOrDefault();
```

### 自定义扩展方法

```cs
public static class CustomExtensions  
{  
    public static IEnumerable<T> CustomFilter<T>(this IEnumerable<T> source, Func<T, bool> predicate)  
    {  
        foreach (var item in source)  
        {  
            if (predicate(item))  
            {  
                yield return item;  
            }  
        }  
    }  
}  
// 使用自定义扩展方法  
var filteredData = students.CustomFilter(s => s.Age > 20);
```

### 示例

> 假设有一个包含学生信息的列表，每个学生有姓名、年龄和成绩。使用LINQ查询来选择年龄大于20岁的学生，然后按照他们的成绩进行分组，并选择每个分组中年龄最小的学生的姓名。

```cs
// 示例数据源  
List<Student> students = new List<Student>  
{  
    new Student { Name = "Alice", Age = 25, Grade = "A" },  
    new Student { Name = "Bob", Age = 30, Grade = "B" },  
    new Student { Name = "Barry", Age = 35, Grade = "C" },  
    new Student { Name = "Charlie", Age = 22, Grade = "A" },  
    new Student { Name = "David", Age = 21, Grade = "C" },  
    new Student { Name = "Damon", Age = 28, Grade = "B" },  
    new Student { Name = "Echo", Age = 18, Grade = "C" }  
};  
  
// 使用LINQ进行查询  
var result = students  
    .Where(student => student.Age > 20) // WHERE: 选择年龄大于20的学生  
    .GroupBy(student => student.Grade)  // GROUP BY: 按成绩分组  
    .Select(group => group.OrderBy(student => student.Age).First().Name) // SELECT: 选择每个分组中年龄最小的学生的姓名  
    .ToList(); //转换为List<Student>()  

//输出结果  
["Charlie","Damon","David"]
```



## 语法糖

> 语法糖需要根据`c#`版本来确实是否可以使用，一般情况下`c# 8.0`及以上的`C#`版本都已支持。

### 对象判空
```c#
// 判断对象是否为空，为空抛出异常
if(obj == null) throw new NullReferenceException();

// 简化的语法糖
obj ?? throw new NullReferenceException();
```

### 对象为空赋值
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

### 可空类型判空并赋值
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

### 减少空引用
```c#
// 判断数组或list不能null且有元素
if(list != null && list.Count > 0)

// 简化的语法糖 当list为null时，将直接返回false
if(list?.Count > 0)

// 同样可运用在赋值时，如果obj为null，将不会取obj.text的值，而是将会为text赋值null
string text = obj?.text;
```

### 判断参数类型并转换类型+校验
```c#
// 判断value是否为 string 类型，如果value是 string 类型
// 那么将value转换为 string 类型，并赋值给 stringValue
// 再判断 stringValue是否不为Null或空
if(value is string stringValue && !string.IsNullOrEmpty(stringValue))
```

### 切片操作
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

### Switch
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

