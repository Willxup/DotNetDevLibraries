<p align="center">
  <a href="../en/Office.md"><strong>English</strong></a> ｜ <a href="../cn/Office.md">简体中文</a>
</p>

[← Contents](../../README.md)

# MiniExcel

## Introduction

`MiniExcel` is a simple, efficient .NET tool for reading, writing, and filling Excel data, designed to help avoid out-of-memory errors. Its open-source NuGet package is small and fast, and it works well for simple Excel imports and exports.

`GitHub`: [![GitHub Release](https://img.shields.io/github/v/release/mini-software/MiniExcel)](https://github.com/mini-software/MiniExcel)

`NuGet`: [![NuGet Version](https://img.shields.io/nuget/v/MiniExcel)](https://www.nuget.org/packages/MiniExcel)


## Quick Start

### NuGet Packages

```c#
MiniExcel
```

### Basic Usage

- Create a model and use the `ExcelColumnName` attribute to mark the Excel column names
```c#
public class TestModel
{
	/// <summary>
	/// ID
	/// </summary>
	[ExcelColumnName("编号")] // Column name: ID.
	public string Id { get; set; }
	/// <summary>
	/// Name
	/// </summary>
	[ExcelColumnName("名称")] // Column name: Name.
	public string Name { get; set; }
}
```

- Read the file from a `stream`; `MiniExcel` maps the data to the configured model
```c#
public async Task ImportMajorByExcel(IFormFile file)
{
	// Get a stream from the file
	using Stream steam = file.OpenReadStream();

	// Read Excel data from the stream
	var rows = await steam.QueryAsync<ImportMajorModel>(excelType: ExcelType.XLSX);
}
```

`MiniExcel` is easy to use; see its documentation on GitHub for other features. For very complex spreadsheet exports, `MiniExcel` may not be enough.

# NPOI

## Introduction

`NPOI` is the .NET version of the [Apache POI](https://poi.apache.org/) project. It makes it easy to read and write Office 2003/2007 files. It also offers flexible Excel handling, so you can try it for more complex spreadsheet exports.

`GitHub`: [![GitHub Release](https://img.shields.io/github/v/release/nissl-lab/npoi)](https://github.com/nissl-lab/npoi)

`NuGet`: [![NuGet Version](https://img.shields.io/nuget/v/NPOI)](https://www.nuget.org/packages/NPOI)

## Quick Start

The following code exports an Excel file with dropdown list validation. The related helper methods are in [NPOI Helper Source](#npoi-helper-source).
```c#
/// <summary>
/// Export an Excel template
/// </summary>
/// <returns></returns>
public async Task<byte[]> ExportExcelTemplate()
{
	// Create an Excel workbook
	XSSFWorkbook workbook = new();
	XSSFSheet sheet = (XSSFSheet)workbook.CreateSheet($"导入模板"); // Sheet name: Import template.

	// Set the title font size
	var titleFontStyle = NPOIExcelHelper.GetFontStyle(workbook, "Arial", 10, true);

	// Set the title styles
	var titleStyle = NPOIExcelHelper.GetTitleCellStyle(workbook, titleFontStyle);
	var titleStyleWithColor = NPOIExcelHelper.GetTitleCellStyle(workbook, titleFontStyle, new XSSFColor(new byte[] { 255, 192, 0 }));

	// Title cell information
	ICellStyle[] cellStyleList = new ICellStyle[2] { titleStyle, titleStyleWithColor }; // 0: title without color; 1: title with color
	Dictionary<string, int> cellInfoDic = new() // Column names: Test name 1 through Test name 6.
	{
		{ "测试名称1", 1 },{ "测试名称2", 0 },{ "测试名称3", 1 },{ "测试名称4", 1 },{ "测试名称5", 1 },{ "测试名称6", 0 }
	};

	// Set the titles
	NPOIExcelHelper.SetTitleRow(sheet, NPOIExcelHelper.GetExcelTemplateTitleRow(cellInfoDic, cellStyleList), NPOIExcelHelper.GetDefaultStyle(workbook));


	// Set the dropdown data source (data longer than 255)
	List<string> codeList = new() {"test1", "test2", "test3" }

	NPOIExcelHelper.SetMoreDropDownList(sheet, codeList, 0, "测试下拉数据源"); // Sheet name: Test dropdown data source.

	// Set the dropdown data (data shorter than 255)
	List<string> enrollTypeList = new()
	{ "测试1", "测试2", "测试3" }; // Options: Test 1, Test 2, Test 3.
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
			throw new Exception("没有可以导出的数据!"); // No data to export.
	}
	catch (Exception ex)
	{
		throw new Exception($"导出数据失败!"); // Data export failed.
	}
}
```

## NPOI Helper Source

### Style Settings

- Get the default style
```c#
/// <summary>
/// Get the default style
/// </summary>
/// <param name="workbook"></param>
/// <returns></returns>
public static XSSFCellStyle GetDefaultStyle(IWorkbook workbook)
{
	XSSFCellStyle style = (XSSFCellStyle)workbook.CreateCellStyle();

	// Set the cell style
	style.Alignment = HorizontalAlignment.Justify; // Justify (wrap text automatically)
	style.VerticalAlignment = VerticalAlignment.Center; // Center vertically

	// Set the cell format
	IDataFormat dataformat = workbook.CreateDataFormat();
	style.DataFormat = dataformat.GetFormat("@"); // dataformat.GetFormat("test") can also be used

	// Set the cell font
	style.SetFont(GetFontStyle(workbook, "Arial", 10, true));

	return style;
}
```

 - Get the font style
```c#
/// <summary>
/// Get the font style
/// </summary>
/// <param name="workbook">Workbook</param>
/// <param name="fontName">Font name</param>
/// <param name="point">Font size</param>
/// <param name="isBold">Whether to use bold</param>
/// <returns></returns>
public static IFont GetFontStyle(IWorkbook workbook, string fontName, short point, bool isBold = false)
{
	XSSFFont font = (XSSFFont)workbook.CreateFont();
	font.FontName = fontName; // Font name
	font.FontHeightInPoints = point; // Font size
	if (isBold)
		font.IsBold = true; // Whether to use bold
	return font;
}
```

- Get the title cell style
```c#
/// <summary>
/// Get the title cell style
/// </summary>
/// <param name="workbook">Workbook</param>
/// <param name="fontStyle">Font style</param>
/// <param name="color">Color</param>
/// <returns></returns>
public static XSSFCellStyle GetTitleCellStyle(IWorkbook workbook, IFont fontStyle, XSSFColor color = null)
{
	XSSFCellStyle titleStyle = (XSSFCellStyle)workbook.CreateCellStyle();

	// Set the cell style
	titleStyle.Alignment = HorizontalAlignment.Center; // Center horizontally
	titleStyle.VerticalAlignment = VerticalAlignment.Center; // Center vertically

	// Set the cell format
	IDataFormat dataformat = workbook.CreateDataFormat();
	titleStyle.DataFormat = dataformat.GetFormat("@"); // dataformat.GetFormat("text") can also be used

	// Set the cell font
	titleStyle.SetFont(fontStyle);

	// Set the fill color
	if (color != null)
	{
		titleStyle.FillForegroundColorColor = color;
		titleStyle.FillPattern = FillPattern.SolidForeground;
	}

	return titleStyle;
}
```

- Get the normal cell style
```c#
/// <summary>
/// Get the normal cell style
/// </summary>
/// <param name="workbook">Workbook</param>
/// <param name="font"></param>
/// <returns></returns>
public static XSSFCellStyle GetNormalCellStyle(IWorkbook workbook, IFont font)
{
	XSSFCellStyle cellStyle = (XSSFCellStyle)workbook.CreateCellStyle();

	// Set the cell style
	cellStyle.Alignment = HorizontalAlignment.Justify; // Justify (wrap text automatically)
	cellStyle.VerticalAlignment = VerticalAlignment.Center; // Center vertically

	// Set the cell format
	IDataFormat dataformat = workbook.CreateDataFormat();
	cellStyle.DataFormat = dataformat.GetFormat("@"); // dataformat.GetFormat("text") can also be used

	// Set the cell font
	cellStyle.SetFont(font);

	return cellStyle;
}
```

### Title Cell Settings

- Set the title row
```c#
/// <summary>
/// Set the title row
/// </summary>
/// <param name="sheet">Sheet</param>
/// <param name="cellInfoList">Title cells</param>
/// <param name="columnStyle">Column style</param>
/// <param name="heightInPoint">Cell height (default: 27)</param>
/// <param name="columnWidth">Cell width (default: 30 * 256 [15 characters])</param>
public static void SetTitleRow(ISheet sheet, List<NPOIExcelTitleCellModel> cellInfoList, ICellStyle columnStyle, float heightInPoint = 27, int columnWidth = 15)
{
	// Create a row
	IRow title = sheet.CreateRow(0);
	// Set the row height
	title.HeightInPoints = heightInPoint;

	// Set each title
	if (cellInfoList?.Count > 0)
	{
		for (int i = 0; i < cellInfoList.Count; i++)
		{
			// Get the cell settings
			var cellInfo = cellInfoList[i];

			// Set the default column style
			sheet.SetDefaultColumnStyle(i, columnStyle);

			// Create a cell and set its value
			title.CreateCell(i).SetCellValue(cellInfo.TitleCellName);

			// Set the cell style
			title.GetCell(i).CellStyle = cellInfo.TitleCellStyle;

			// Widen the cell if the title contains the Chinese word for "project"
			if (cellInfo.TitleCellName.Contains("项目"))
			{
				sheet.SetColumnWidth(i, (columnWidth + 7) * 256);
			}
			// For titles over 10 characters, add 5 characters of width per 3 extra characters, rounding up
			else if (cellInfo.TitleCellName.Length > 10)
			{
				sheet.SetColumnWidth(i, (columnWidth + ((cellInfo.TitleCellName.Length - 10) / 3 + 1) * 5) * 256);
			}
			// Other titles
			else
			{
				sheet.SetColumnWidth(i, columnWidth * 256);
			}
		}
	}
}
```

- Get the template title row
```c#
/// <summary>
/// Get the template title row
/// </summary>
/// <param name="cellInfoDic">Cell content dictionary (cell content: style index)</param>
/// <param name="cellStyleList">List of styles</param>
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
				TitleCellName = cellInfo.Key, // Key is the cell content
				TitleCellStyle = cellStyleList[cellInfo.Value] // Value is the cell style
			};
			titleCellList.Add(cell);
		}
	}
	return titleCellList;
}
```

- Title style model
```c#
public class NPOIExcelTitleCellModel
{
	/// <summary>
	/// Title cell name
	/// </summary>
	public string TitleCellName { get; set; }
	/// <summary>
	/// Title cell style
	/// </summary>
	public ICellStyle TitleCellStyle { get; set; }
}
```

### Content Cell Settings

```c#
/// <summary>
/// Set the content rows
/// </summary>
/// <typeparam name="T"></typeparam>
/// <param name="sheet"></param>
/// <param name="contentList"></param>
/// <param name="cellStyle"></param>
public static void SetContentRows<T>(ISheet sheet, List<T> contentList, ICellStyle cellStyle) where T : class, new()
{
	if (contentList?.Count > 0)
	{
		// Get all properties of the generic type
		PropertyInfo[] propertyInfos = typeof(T).GetProperties();

		// Row number starts at 1 (0 is the title)
		int rowNum = 1;
		foreach (var item in contentList)
		{
			// Create a row
			IRow row = sheet.CreateRow(rowNum);
			// Cell number
			int cellNum = 0;

			// Set values using reflection
			foreach (PropertyInfo p in propertyInfos)
			{
				// Create a cell and set its content
				row.CreateCell(cellNum).SetCellValue(p.GetValue(item).ToString());

				// Set the cell style
				row.GetCell(cellNum).CellStyle = cellStyle;

				cellNum++;
			}
			rowNum++;
		}
	}
}
```

### Dropdown List Settings

- Set a dropdown list (less than 255)
```c#
/// <summary>
/// Set a dropdown list (less than 255)
/// </summary>
/// <param name="sheet">Current sheet</param>
/// <param name="dropDownList">Dropdown list</param>
/// <param name="column">Column containing dropdown cells/param>
/// <param name="startRow">First row of dropdown cells (default: 1)</param>
/// <param name="lastRow">Last row of dropdown cells (default: 65535)</param>
public static void SetLessDropDownList(XSSFSheet sheet, string[] dropDownList, int column, int startRow = 1, int lastRow = 65535)
{
	// Data validation helper
	XSSFDataValidationHelper dvHelper = new(sheet);

	// Create the data to validate
	XSSFDataValidationConstraint dvConstraint = (XSSFDataValidationConstraint)dvHelper.CreateExplicitListConstraint(dropDownList);

	// Cell range to validate
	CellRangeAddressList addressList = new(startRow, lastRow, column, column);

	// Create the data validation
	XSSFDataValidation validation = (XSSFDataValidation)dvHelper.CreateValidation(dvConstraint, addressList);

	// Add the data validation
	sheet.AddValidationData(validation);
}
```

- Set a dropdown list (more than 255)
```c#
/// <summary>
/// Set a dropdown list (more than 255)
/// </summary>
/// <param name="sheet">Current sheet</param>
/// <param name="dropDownList">Dropdown list</param>
/// <param name="column">Column containing dropdown cells/param>
/// <param name="sheetName">Sheet name (default: HiddenSheet)</param>
/// <param name="startRow">First row of dropdown cells (default: 1)</param>
/// <param name="lastRow">Last row of dropdown cells (default: 65535)</param>
public static void SetMoreDropDownList(XSSFSheet sheet, List<string> dropDownList, int column, string sheetName = "HiddenSheet", int startRow = 1, int lastRow = 65535)
{
	if (sheet == null) return;

	IWorkbook workbook = sheet.Workbook;

	// Create a data validation sheet
	ISheet hiddenSheet = workbook.CreateSheet(sheetName); ;

	// Fill the data validation sheet
	for (int i = 0, length = dropDownList.Count; i < length; i++)
	{
		string value = dropDownList[i];
		IRow row = hiddenSheet.CreateRow(i);
		ICell cell = row.CreateCell(0);
		cell.SetCellValue(value);
	}

	// Create the cell range
	IName namedCell = workbook.CreateName();
	namedCell.NameName = sheetName;
	namedCell.RefersToFormula = sheetName + "!$A$1:$A$" + dropDownList.Count;

	// Data validation helper
	XSSFDataValidationHelper dvHelper = new(sheet);

	// Create the data to validate
	XSSFDataValidationConstraint dvConstraint = (XSSFDataValidationConstraint)dvHelper.CreateFormulaListConstraint(sheetName);

	// Cell range to validate
	CellRangeAddressList addressList = new(startRow, lastRow, column, column);

	// Create the data validation
	XSSFDataValidation validation = (XSSFDataValidation)dvHelper.CreateValidation(dvConstraint, addressList);

	// Get the index of the data validation sheet
	int hiddenSheetIndex = workbook.GetSheetIndex(hiddenSheet);

	// Hide the data validation sheet
	workbook.SetSheetHidden(hiddenSheetIndex, SheetState.Hidden);

	// Add the data validation
	sheet.AddValidationData(validation);
}
```
