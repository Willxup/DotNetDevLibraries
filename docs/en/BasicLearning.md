<p align="center">
  <a href="../en/BasicLearning.md"><strong>English</strong></a> ｜ <a href="../cn/BasicLearning.md">简体中文</a>
</p>

[← Contents](../../README.md)

# Contents

- [Official Resources](#official-resources)
- [Development Tips](#development-tips)
  - [LINQ](#linq)
  - [Syntax Shortcuts](#syntax-shortcuts)


# Official Resources

Official documentation is the most reliable source. Microsoft's machine translations can be rough, but try the official docs first. Look elsewhere if they don't answer your question.

- [C# official documentation (Chinese)](https://learn.microsoft.com/zh-cn/dotnet/csharp/)
- [Run C# code online (Chinese)](https://learn.microsoft.com/zh-cn/dotnet/csharp/tour-of-csharp/#code-try-0)
- [.NET official documentation (Chinese)](https://learn.microsoft.com/zh-cn/dotnet/fundamentals/)
- [Download .NET (Chinese)](https://dotnet.microsoft.com/zh-cn/download/dotnet)

# Development Tips

These practical tips can help you write less code and work more efficiently.

## LINQ

> LINQ (Language-Integrated Query) is a query language in C#. It provides a unified way to query data more clearly and concisely.

Official documentation: [https://learn.microsoft.com/zh-cn/dotnet/csharp/linq/](https://learn.microsoft.com/zh-cn/dotnet/csharp/linq/)

### FROM

> Any data source, including object collections, databases, and XML.

### WHERE

> Filter results.

```cs
// Sample data source
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

// Use WHERE to select students with grade A
var result1 = students.Where(student => student.Grade == "A");

// Use WHERE to select students over age 20
var result2 = students.Where(student => student.Age > 20);

// Use WHERE to select students whose names contain 'D'
var result3 = students.Where(student => student.Name.Contains("D"));

// Use WHERE to select students named 'Bob' or 'Echo'
List<string> nameList = new() { "Bob", "Echo" };
var result4 = students.Where(student => nameList.Contains(student.Name));
```

### SELECT

> Select results.

```cs
// Sample data source
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

// Use SELECT to create a collection of anonymous objects, often with Where
var result1 = students.Select(student =>
    new
    {
        student.Name,
        student.Age
    });

// Use SELECT to create a collection of objects of a specified class
var result2 = students.Select(student => new StudentDto()
    {
        StudentName = student.Name,
        StudentAge = student.Age
    });
```

### GROUPBY

> Group results.

```cs
// Sample data source
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

// Use GROUP BY to group students by grade
var groupedByGrade = students.GroupBy(student => student.Grade);
```

### ORDERBY

> Sort results.

```cs
// Sample data source
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

// Use LINQ OrderBy to sort
var result1 = students.OrderBy(student => student.Age);

// Use LINQ OrderByDescending to sort in descending order
var result2 = students.OrderByDescending(student => student.Age);
```

### JOIN

> `Join`: Performs an inner join, returning matching elements from both data sources.
>
> `GroupJoin`: Performs a left outer join, returning every element from the left data source with a nested collection of its matches from the right data source.

```cs
// Sample data source
List<Department> departments = new List<Department>
{
    new Department { ID = 1, Name = "HR" },
    new Department { ID = 2, Name = "IT" }
};
// Sample data source
List<Employee> employees = new List<Employee>
{
    new Employee { DepartmentID = 1, Name = "Alice" },
    new Employee { DepartmentID = 2, Name = "Bob" },
    new Employee { DepartmentID = 1, Name = "Charlie" },
    new Employee { DepartmentID = 3, Name = "David" }
};

// Use Join to combine departments and employees into a collection of their names
var joinQuery = departments
    .Join(employees,
        department => department.ID,
        employee => employee.DepartmentID,
        (department, employee) => new { Department = department.Name, Employee = employee.Name }
    );

// Use GroupJoin to return all departments and a nested collection of employees for each department
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

### Convert Results

```cs
// ToList(): Convert the results to a List.
List<Student> resultList = result.ToList();

// ToDictionary(): Convert the results to a Dictionary.
Dictionary<string, int> resultDictionary = students
    .ToDictionary(student => student.Name, student => student.Age);

// ToArray(): Convert the results to an array.
Student[] resultArray = result.ToArray();

// First(): Get the first result.
Student firstStudent = result.First();

// FirstOrDefault(): Get the first result, or a default value if there are no results.
Student firstStudent = result.FirstOrDefault();
```

### Custom Extension Methods

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
// Use the custom extension method
var filteredData = students.CustomFilter(s => s.Age > 20);
```

### Example

> Suppose a list contains students with names, ages, and grades. Use LINQ to select students over age 20, group them by grade, and select the name of the youngest student in each group.

```cs
// Sample data source
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

// Query with LINQ
var result = students
    .Where(student => student.Age > 20) // WHERE: Select students over age 20
    .GroupBy(student => student.Grade)  // GROUP BY: Group by grade
    .Select(group => group.OrderBy(student => student.Age).First().Name) // SELECT: Select the name of the youngest student in each group
    .ToList(); // Convert to List<string>()

// Output
["Charlie","Damon","David"]
```



## Syntax Shortcuts

> Check your `c#` version to see which shortcuts are available. In general, `c# 8.0` and later versions of `C#` support them.

### Check an Object for Null
```c#
// Check whether the object is null and throw an exception if it is
if(obj == null) throw new NullReferenceException();

// Shorter syntax
obj ?? throw new NullReferenceException();
```

### Assign a Value if an Object Is Null
```c#
// Assign a new value only if the object is null
// Do not assign a value if the object is not null
if(obj == null)
{
	obj = new object();
}

// Shorter syntax
obj ??= new object();
```

### Check a Nullable Value and Assign a Value
```c#
// Nullable type
int? nums = null;

// Assign a different value depending on whether the value is null
if(nums == null)
{
	result = -1;
}
else
{
	result = nums;
}

// Shorter syntax
int result = nums ?? -1;
```

### Reduce Null References
```c#
// Check that the array or list is not null and has elements
if(list != null && list.Count > 0)

// Shorter syntax: return false directly when list is null
if(list?.Count > 0)

// This also works for assignment: if obj is null, text is set to null without reading obj.text
string text = obj?.text;
```

### Check, Convert, and Validate a Parameter
```c#
// Check whether value is a string
// If so, convert it to a string and assign it to stringValue
// Then check that stringValue is not null or empty
if(value is string stringValue && !string.IsNullOrEmpty(stringValue))
```

### Slices
```c#
// **The numbers in [] below are indexes**
// **A range with an end index does not include the element at that index**

// Sample array
string[] arr = new string[] { "10", "20", "30", "40", "50", "60", "70", "80", "90", "100" };

// Get the last element
string str = arr[^1];

// Get the first 3 elements, from index 0 to index 3 (exclusive): ["10","20","30"]
// Index 0 can be omitted to start at the beginning and end before index 3
// string[] strs = arr[..3];
string[] strs1 = arr[0..3];

// Get the last 3 elements, from the third element from the end through the end: ["80", "90", "100"]
// The final index is omitted: string[] strs21 = arr[^3..^0];
// ^0 points to the position after the last element, which does not exist as an element
string[] strs2 = arr[^3..];

// Get a range of elements using indexes from the start
// From index 3 to index 7 (exclusive): ["40", "50", "60", "70"]
string[] strs3 = arr[3..7];

// Get a range of elements using indexes from the end
// From the fourth element from the end to the second (exclusive): ["70","80"]
string[] strs4 = arr[^4..^2];
```

### Switch
```c#
public string GetNums(int num)
{
	// With this switch syntax, the return types must match
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

