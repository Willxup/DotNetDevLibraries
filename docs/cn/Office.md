# MiniExcel

## 介绍

`MiniExcel`简单、高效避免`OOM`的`.NET`处理`Excel`查、写、填充数据工具。这个开源项目`nuget`包很小，并且速度也很快。简单的`Excel`导出导入非常好用。

`Github`：[![GitHub Release](https://img.shields.io/github/v/release/mini-software/MiniExcel)](https://github.com/mini-software/MiniExcel)

`Nuget`：[![NuGet Version](https://img.shields.io/nuget/v/MiniExcel)](https://www.nuget.org/packages/MiniExcel)


## 快速上手

### Nuget包

```c#
MiniExcel
```

### 简单使用

- 创建一个模型，`Excel`中的字段名使用`ExcelColumnName`特性标注
```c#
public class TestModel
{
	/// <summary>
	/// 编号
	/// </summary>
	[ExcelColumnName("编号")]
	public string Id { get; set; }
	/// <summary>
	/// 名称
	/// </summary>
	[ExcelColumnName("名称")]
	public string Name { get; set; }
}
```

- 直接从`stream`中获取文件，`MiniExcel`将会自动映射为设置的模型
```c#
public async Task ImportMajorByExcel(IFormFile file)
{
	//从文件中获取流
	using Stream steam = file.OpenReadStream();

	//从流中获取Excel数据
	var rows = await steam.QueryAsync<ImportMajorModel>(excelType: ExcelType.XLSX);
}
```

`MiniExcel`使用起来非常的简单，其他功能可以参考`Github`上的文档。但是，如果需要导出非常复杂的表格时，`MiniExcel`就无法胜任了。

# NPOI

## 介绍

`NPOI`是 [`Apache POI `](https://poi.apache.org/)项目的` .NET` 版本。使用`NPOI`，您可以非常轻松地读取/写入`Office 2003/2007` 文件。它可以非常灵活的操作`Excel`，对于复杂的`Excel`生成导出，可以试一试它。

`Github`：[![GitHub Release](https://img.shields.io/github/v/release/nissl-lab/npoi)](https://github.com/nissl-lab/npoi)

`Nuget`：[![NuGet Version](https://img.shields.io/nuget/v/NPOI)](https://www.nuget.org/packages/NPOI)

## 快速上手

下面是导出一个拥有下拉框数据源校验的`Excel`文件的源码，所有相关的封装方法都在[NPOI封装源码](#NPOI封装源码)中。
```c#
/// <summary>
/// 导出EXCEL模板
/// </summary>
/// <returns></returns>
public async Task<byte[]> ExportExcelTemplate()
{
	//创建Excel
	XSSFWorkbook workbook = new();
	XSSFSheet sheet = (XSSFSheet)workbook.CreateSheet($"导入模板");

	//设置标题字体大小
	var titleFontStyle = NPOIExcelHelper.GetFontStyle(workbook, "Arial", 10, true);

	//设置标题样式
	var titleStyle = NPOIExcelHelper.GetTitleCellStyle(workbook, titleFontStyle);
	var titleStyleWithColor = NPOIExcelHelper.GetTitleCellStyle(workbook, titleFontStyle, new XSSFColor(new byte[] { 255, 192, 0 }));

	//标题单元格信息
	ICellStyle[] cellStyleList = new ICellStyle[2] { titleStyle, titleStyleWithColor }; //0不带颜色得标题 1带颜色的标题
	Dictionary<string, int> cellInfoDic = new()
	{
		{ "测试名称1", 1 },{ "测试名称2", 0 },{ "测试名称3", 1 },{ "测试名称4", 1 },{ "测试名称5", 1 },{ "测试名称6", 0 }
	};

	//设置标题
	NPOIExcelHelper.SetTitleRow(sheet, NPOIExcelHelper.GetExcelTemplateTitleRow(cellInfoDic, cellStyleList), NPOIExcelHelper.GetDefaultStyle(workbook));


	//设置下拉数据源(数据大小大于255长度)
	List<string> codeList = new() {"test1", "test2", "test3" }

	NPOIExcelHelper.SetMoreDropDownList(sheet, codeList, 0, "测试下拉数据源");

	//设置下拉数据(数据大小小于255长度)
	List<string> enrollTypeList = new()
	{ "测试1", "测试2", "测试3" };
	NPOIExcelHelper.SetLessDropDownList(sheet, enrollTypeList.ToArray(), 4);

	try
	{
		if (workbook != null)
		{
			byte[] buffer = new byte[1024 * 2];
			using (MemoryStream ms = new())
			{
				workbook.Write(ms);
				buffer = ms.ToArray();
				ms.Close();
			}
			return buffer;
		}
		else
			throw new Exception("没有可以导出的数据!");
	}
	catch (Exception ex)
	{
		throw new Exception($"导出数据失败!");
	}
}
```

## NPOI封装源码

### 样式设置

- 获取默认样式
```c#
/// <summary>
/// 获取默认样式
/// </summary>
/// <param name="workbook"></param>
/// <returns></returns>
public static XSSFCellStyle GetDefaultStyle(IWorkbook workbook)
{
	XSSFCellStyle style = (XSSFCellStyle)workbook.CreateCellStyle();

	//设置单元格样式
	style.Alignment = HorizontalAlignment.Justify; //两端自动对齐（自动换行）
	style.VerticalAlignment = VerticalAlignment.Center; //垂直居中对齐

	//设置单元格格式
	IDataFormat dataformat = workbook.CreateDataFormat(); 
	style.DataFormat = dataformat.GetFormat("@"); //dataformat.GetFormat("test") 两种方式都可以

	//设置单元格字体
	style.SetFont(GetFontStyle(workbook, "Arial", 10, true));

	return style;
}
```

 - 获取字体样式
```c#
/// <summary>
/// 获取字体样式
/// </summary>
/// <param name="workbook">工作簿</param>
/// <param name="fontName">字体名称</param>
/// <param name="point">字体大小</param>
/// <param name="isBold">是否加粗</param>
/// <returns></returns>
public static IFont GetFontStyle(IWorkbook workbook, string fontName, short point, bool isBold = false)
{
	XSSFFont font = (XSSFFont)workbook.CreateFont();
	font.FontName = fontName; //字体名称
	font.FontHeightInPoints = point; //字体大小
	if (isBold)
		font.IsBold = true; //是否加粗
	return font;
}
```

-  获取标题单元格样式
```c#
/// <summary>
/// 获取标题单元格样式
/// </summary>
/// <param name="workbook">工作簿</param>
/// <param name="fontStyle">字体样式</param>
/// <param name="color">颜色</param>
/// <returns></returns>
public static XSSFCellStyle GetTitleCellStyle(IWorkbook workbook, IFont fontStyle, XSSFColor color = null)
{
	XSSFCellStyle titleStyle = (XSSFCellStyle)workbook.CreateCellStyle();
	
	//设置单元格样式
	titleStyle.Alignment = HorizontalAlignment.Center; //两端居中对齐
	titleStyle.VerticalAlignment = VerticalAlignment.Center; //垂直居中对齐

	//设置单元格格式
	IDataFormat dataformat = workbook.CreateDataFormat();
	titleStyle.DataFormat = dataformat.GetFormat("@"); //dataformat.GetFormat("text") 两种方式都可以

	//设置单元格字体
	titleStyle.SetFont(fontStyle);

	//设置填充颜色
	if (color != null)
	{
		titleStyle.FillForegroundColorColor = color;
		titleStyle.FillPattern = FillPattern.SolidForeground;
	}

	return titleStyle;
}
```

- 获取普通单元格样式
```c#
/// <summary>
/// 获取普通单元格样式
/// </summary>
/// <param name="workbook">工作簿</param>
/// <param name="font"></param>
/// <returns></returns>
public static XSSFCellStyle GetNormalCellStyle(IWorkbook workbook, IFont font)
{
	XSSFCellStyle cellStyle = (XSSFCellStyle)workbook.CreateCellStyle();

	//设置单元格样式
	cellStyle.Alignment = HorizontalAlignment.Justify; //两端自动对齐（自动换行）
	cellStyle.VerticalAlignment = VerticalAlignment.Center; //垂直居中对齐

	//设置单元格格式
	IDataFormat dataformat = workbook.CreateDataFormat();
	cellStyle.DataFormat = dataformat.GetFormat("@"); //dataformat.GetFormat("text") 两种方式都可以

	//设置单元格字体
	cellStyle.SetFont(font);

	return cellStyle;
}
```

### 标题单元格设置

- 设置标题行
```c#
/// <summary>
/// 设置标题行
/// </summary>
/// <param name="sheet">表</param>
/// <param name="cellInfoList">标题单元格</param>
/// <param name="columnStyle">列样式</param>
/// <param name="heightInPoint">单元格高度(默认27)</param>
/// <param name="columnWidth">单元格宽度(默认30 * 256[15个字符])</param>
public static void SetTitleRow(ISheet sheet, List<NPOIExcelTitleCellModel> cellInfoList, ICellStyle columnStyle, float heightInPoint = 27, int columnWidth = 15)
{
	//创建行
	IRow title = sheet.CreateRow(0);
	//设置行高
	title.HeightInPoints = heightInPoint;

	//循环设置标题
	if (cellInfoList?.Count > 0)
	{
		for (int i = 0; i < cellInfoList.Count; i++)
		{
			//获取单元格设置
			var cellInfo = cellInfoList[i];

			//设置列默认样式
			sheet.SetDefaultColumnStyle(i, columnStyle);

			//创建单元格并赋值
			title.CreateCell(i).SetCellValue(cellInfo.TitleCellName);

			//设置单元格样式
			title.GetCell(i).CellStyle = cellInfo.TitleCellStyle;

			//包含项目，加大单元格宽度
			if (cellInfo.TitleCellName.Contains("项目"))
			{
				sheet.SetColumnWidth(i, (columnWidth + 7) * 256);
			}
			//长度大于10字符，按照每超过3字符加5字符单元格宽度 不足3字符当作3字符计算
			else if (cellInfo.TitleCellName.Length > 10)
			{
				sheet.SetColumnWidth(i, (columnWidth + ((cellInfo.TitleCellName.Length - 10) / 3 + 1) * 5) * 256);
			}
			//其他
			else
			{
				sheet.SetColumnWidth(i, columnWidth * 256);
			}
		}
	}
}
```

- 获取模板标题行
```c#
/// <summary>
/// 获取模板标题行
/// </summary>
/// <param name="cellInfoDic">单元格内容字典(单元格内容：样式索引)</param>
/// <param name="cellStyleList">样式列表</param>
/// <returns></returns>
public static List<NPOIExcelTitleCellModel> GetExcelTemplateTitleRow(Dictionary<string, int> cellInfoDic, ICellStyle[] cellStyleList)
{
	List<NPOIExcelTitleCellModel> titleCellList = null;
	if (cellInfoDic?.Count > 0 && cellStyleList?.Length > 0)
	{
		titleCellList = new List<NPOIExcelTitleCellModel>();

		foreach (var cellInfo in cellInfoDic)
		{
			NPOIExcelTitleCellModel cell = new()
			{
				TitleCellName = cellInfo.Key, //key为单元格内容
				TitleCellStyle = cellStyleList[cellInfo.Value] //value为单元格样式
			};
			titleCellList.Add(cell);
		}
	}
	return titleCellList;
}
```

- 标题样式模型
```c#
public class NPOIExcelTitleCellModel
{
	/// <summary>
	/// 标题单元格名称
	/// </summary>
	public string TitleCellName { get; set; }
	/// <summary>
	/// 标题单元格样式
	/// </summary>
	public ICellStyle TitleCellStyle { get; set; }
}
```

### 内容单元格设置

```c#
/// <summary>
/// 设置内容行
/// </summary>
/// <typeparam name="T"></typeparam>
/// <param name="sheet"></param>
/// <param name="contentList"></param>
/// <param name="cellStyle"></param>
public static void SetContentRows<T>(ISheet sheet, List<T> contentList, ICellStyle cellStyle) where T : class, new()
{
	if (contentList?.Count > 0)
	{
		//获取泛型的所有属性
		PropertyInfo[] propertyInfos = typeof(T).GetProperties();

		//行号，从1开始（0为标题）
		int rowNum = 1;
		foreach (var item in contentList)
		{
			//创建行
			IRow row = sheet.CreateRow(rowNum);
			//单元格序号
			int cellNum = 0;

			//循环反射赋值
			foreach (PropertyInfo p in propertyInfos)
			{
				//创建单元格，设置单元格内容
				row.CreateCell(cellNum).SetCellValue(p.GetValue(item).ToString());

				//设置单元格样式
				row.GetCell(cellNum).CellStyle = cellStyle;

				cellNum++;
			}
			rowNum++;
		}
	}
}
```

### 下拉列表设置

- 设置下拉列表(少于255)
```c#
/// <summary>
/// 设置下拉列表(少于255)
/// </summary>
/// <param name="sheet">当前表</param>
/// <param name="dropDownList">下拉列表</param>
/// <param name="column">需要下拉列表的单元格所在列/param>
/// <param name="startRow">需要下拉列表的单元格起始行(默认1)</param>
/// <param name="lastRow">需要下拉列表的单元格结束行(默认65535)</param>
public static void SetLessDropDownList(XSSFSheet sheet, string[] dropDownList, int column, int startRow = 1, int lastRow = 65535)
{
	//数据校验帮助类
	XSSFDataValidationHelper dvHelper = new(sheet);

	//创建需要校验的数据
	XSSFDataValidationConstraint dvConstraint = (XSSFDataValidationConstraint)dvHelper.CreateExplicitListConstraint(dropDownList);

	//被数据校验的单元格范围
	CellRangeAddressList addressList = new(startRow, lastRow, column, column);

	//创建数据校验
	XSSFDataValidation validation = (XSSFDataValidation)dvHelper.CreateValidation(dvConstraint, addressList);

	//添加数据校验
	sheet.AddValidationData(validation);
}
```

- 设置下拉列表(多于255)
```c#
/// <summary>
/// 设置下拉列表(多于255)
/// </summary>
/// <param name="sheet">当前表</param>
/// <param name="dropDownList">下拉列表</param>
/// <param name="column">需要下拉列表的单元格所在列/param>
/// <param name="sheetName">表名(默认HiddenSheet)</param>
/// <param name="startRow">需要下拉列表的单元格起始行(默认1)</param>
/// <param name="lastRow">需要下拉列表的单元格结束行(默认65535)</param>
public static void SetMoreDropDownList(XSSFSheet sheet, List<string> dropDownList, int column, string sheetName = "HiddenSheet", int startRow = 1, int lastRow = 65535)
{
	if (sheet == null) return;

	IWorkbook workbook = sheet.Workbook;

	//创建 数据校验sheet
	ISheet hiddenSheet = workbook.CreateSheet(sheetName); ;

	//循环将数据填入 数据校验sheet 中
	for (int i = 0, length = dropDownList.Count; i < length; i++)
	{
		string value = dropDownList[i];
		IRow row = hiddenSheet.CreateRow(i);
		ICell cell = row.CreateCell(0);
		cell.SetCellValue(value);
	}

	//创建单元格范围
	IName namedCell = workbook.CreateName();
	namedCell.NameName = sheetName;
	namedCell.RefersToFormula = sheetName + "!$A$1:$A$" + dropDownList.Count;

	//数据校验帮助类
	XSSFDataValidationHelper dvHelper = new(sheet);

	//创建需要校验的数据
	XSSFDataValidationConstraint dvConstraint = (XSSFDataValidationConstraint)dvHelper.CreateFormulaListConstraint(sheetName);

	//被数据校验的单元格范围
	CellRangeAddressList addressList = new(startRow, lastRow, column, column);

	//创建数据校验
	XSSFDataValidation validation = (XSSFDataValidation)dvHelper.CreateValidation(dvConstraint, addressList);

	//获取 数据校验sheet 的索引
	int hiddenSheetIndex = workbook.GetSheetIndex(hiddenSheet);

	//设置该 数据校验sheet 为隐藏sheet
	workbook.SetSheetHidden(hiddenSheetIndex, SheetState.Hidden);

	//添加数据校验
	sheet.AddValidationData(validation);
}
```
