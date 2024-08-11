`Excel.dll`放在`Editor`文件夹下
### Excel数据读取
#### 打开Excel表
`FileStream`读取文件流
`IExcelDataReader`从流中读取Excel数据
`DataSet`数据集合类，将Excel数据转存进其中，方便读取
```cs
private static void OpenExcel()
{
	using (FileStream fs = File.Open(Application.dataPath + "/123.xlsx"), FileMode.Open, FileAccess.Read)
	{
		IExcelDataReader excelReader = ExcelReaderFactory.CreateOpenXmlReader(fs);
		DataSet result = excelReader.AsDataSet();
		for(int i = 0; i < result.Tables.Count; i++)
		{
			Debug.Log("表名: " + result.Tables[i].TableName);
			Debug.Log("行数: " + result.Tables[i].Rows.Count);
			Debug.Log("列数: " + result.Tables[i].Coloums.Count);
		}
		fs.Close();
	}
}
```

#### 获取Excel表中单元格的信息
`DataTable`数据表类 表示Excel文件中的一个类
`DataRow`数据行类
```cs
private static void ReadExcel()
{
	using (FileStream fs = File.Open(Application.dataPath + "/123.xlsx"), FileMode.Open, FileAccess.Read)
	{
		IExcelDataReader excelReader = ExcelReaderFactory.CreateOpenXmlReader(fs);
		DataSet result = excelReader.AsDataSet();
		
		for(int i = 0; i < result.Tables.Count; i++)
		{
			//得到其中一张表的具体数据
			DataTable table = result.Tables[i];
			DataRow row;
			for(int j = 0; j < table.Rows.Count; j++)
			{
				row = table.Rows[j];
				for(int k = 0; k < table.Columns.Count; k++)
				{
					Debug.Log(row[k].ToString());
				}
			}
		}
		fs.Close();
	}
}
```

### 制定配表规则
以TowerInfo.xlsx为例：
![image.png](https://s2.loli.net/2024/08/02/u8zVYh6Otk5fPrG.png)

#### 获取指定目录下所有Excel文件
```cs
public class ExcelTool
{
	public static string EXCEL_PATH = Appliaction.dataPath + "/ArtRes/Excel/";
	//数据结构类的存储路径
	public static string DATA_CLASS_PATH = Application.dataPath + "/Scripts/ExcelData/DataClass/";
	//容器类的存储路径
	public static string DATA_CONTAINER_PATH = Application.dataPath + "/Scripts/ExcelData/Container/";
	//二进制数据存储路径
	public static string DATA_BINARY_PATH = Application.streamingAssetsPath + "/Binary/";
	private static void GenerateExcelInfo()
	{
		DirectoryInfo dInfo = Directory.CreateDirectory(EXCEL_PATH);
		FileInfo[] files = dInfo.GetFiles();
		for(int i = 0; i < files.Length; i++)
		{
			if(files[i].Extension != ".xlsx" && files[i].Extension != ".xls")
			continue;

			using(FileSteram fs = files[i].Open(FileMode.Open, FileAccess.Read))
			{
				IExcelDataReader excelReader = ExcelReaderFactory.CreateOpenXmlReader(fs);
				tableConllection = excelReader.AsDataSet().Tables;
				fs.Close();
			}
			foreach(DataTable table in tableConllection)
			{
				Debug.Log(table.TableName);
				//生成数据结构类
				GenerateExcelDataClass(table);
				//生成容器类
				GenerateExcelContainer(table)
				//生成二进制数据
			}
		}
	}
	//生成数据结构类
	private static void GenerateExcelDataClass(DataTable table)
	{
		DataRow rowName = GetVariableNameRow(table);
		DataRow rowType = GetVariableTypeRow(table);
		//判断文件夹是否存在 没有则创建
		if(!Directory.Exists(DATA_CLASS_PATH))
		Directory.CreateDirectory(DATA_CLASS_PATH);
		string str = "public class " + table.TableName + "\n{\n";
		//变量进行字符串拼接
		for(int i = 0; i < table.Columns.Count; i++)
		{
			str += "	public " + rowType[i].ToString() + " " + rowName[i].ToString() + ";\n";
		}
		str += "}"
		File.WriteAllText(DATA_CLASS_PATH + )
	}
	private static DataRow GetVariableNameRow(DataTable table)
	{
		return table.Rows[0];
	}
	private static DataRow GetVariableTypeRow(DataTable table)
	{
		return table.Rows[1];
	}
}
```
#### 生成数据结构类
```cs
private static void GenerateExcelDataClass(DataTable table)
	{
		DataRow rowName = GetVariableNameRow(table);
		DataRow rowType = GetVariableTypeRow(table);
		//判断文件夹是否存在 没有则创建
		if(!Directory.Exists(DATA_CLASS_PATH))
		Directory.CreateDirectory(DATA_CLASS_PATH);
		string str = "public class " + table.TableName + "\n{\n";
		//变量进行字符串拼接
		for(int i = 0; i < table.Columns.Count; i++)
		{
			str += "	public " + rowType[i].ToString() + " " + rowName[i].ToString() + ";\n";
		}
		str += "}"

		File.WriteAllText(DATA_CLASS_PATH + )
	}
	private static DataRow GetVariableNameRow(DataTable table)
	{
		return table.Rows[0];
	}
	private static DataRow GetVariableTypeRow(DataTable table)
	{
		return table.Rows[1];
	}
```
自动生成的`TowerInfo.cs`类
```cs
public class TowerInfo
{
    public int id;
    public string name;
    public int money;
    public int atk;
    public int atkRange;
    public float offsetTime;
    public int nextLev;
    public string imgRes;
    public string res;
    public int atkType;
    public string eff;
}
```


#### 生成数据容器类
```cs
	private static void GenerateExcelContainer(DataTable table)
	{
		int keyIndex = GetKeyIndex(table);
		DataRow rowType = GetVariableTypeRow(table);
		if(!Directory.Exists(DATA_CONTAINER_PATH))
			Directory.CreateDirectory(DATA_CONTAINER_PATH);

			string str = "using Sysetm.Collections.Generic;\n";
			str += "public class " + table.TableName + "Container" + "\n{\n";
			
			str += "    ";
			str += "public Dictionary<" + rowType[keyIndex].ToString() + ", " + table.TableName + ">";
			str += "dataDic = new " + "Dictionary<" + rowType[keyIndex].ToString() + ", " + table.TableName + ">();\n";
			
			str += "}";
			
			File.WriteAllText(DATA_CONTAINER_PATH + table.TableName + "Container.cs", str);
			//刷新Project窗口
			AssetDatabase.Refresh();
	}
	private static int GetKeyIndex(DataTable table)
	{
		DataRow row = table.Rows[2];
		for(int i = 0; i < table.Columns.Count; i++)
		{
			if (row[i] != null && !string.IsNullOrEmpty(row[i].ToString())) 
			{ return i; }
		}
		return 0;
	}
```
生成的容器类`TowerInfoContainer.cs`如下：
```cs
using System.Collections.Generic;
public class TowerInfoContainer
{
    public Dictionary<int, TowerInfo>dataDic = new Dictionary<int, TowerInfo>();
}
```


#### 生成Excel二进制
```cs
	private static void GenerateExcelBinary(DataTable table)
	{
		if(!Directory.Exists(DATA_BINARY_PATH))
		Directory.CreateDirectory(DATA_BINARY_PATH)；
		using(FileStream fs = new FileStream(DATA_BINARY_PATH + table.TableName + ".tang", FileMode.OpenOrCreate, FileAccess.Write)) //文件后缀自己取名
		{
			//-4：前面4行是配置规则，并不是需要记录的数据内容
			fs.Write(BitConverter.GetBytes(table.Rows.Count - 4), 0, 4);
			//存储主键变量名
			string keyName = GetVariableNameRow(table)[GetKeyIndex(table)].ToString();
			bytes[] bytes = Encoding.UTF8.GetBytes(keyName);
			//存储字符串字节数组长度
			fs.Write(BitConverter.GetBytes(bytes.Length), 0, 4);
			//存储字符串字节数组
			fs.Write(bytes, 0, bytes.Length);
			DataRow row;
			DataRow rowType = GetVariableTypeRow(table);
			for(int i = 4 ; i < tables.Rows.Count; i++)
			{
				row = table.Rows[i];
				for(int j = 0; j < table.Columns.Count; j++)
				{
					switch(rowType[j].ToString())
					{
						case "int":
						fs.Write(BitConverter.GetBytes(int.Parse(row[j].ToString()), 0, 4));
						break;
						case "float"
						fs.Write(BitConverter.GetBytes(float.Parse(row[j].ToString()), 0, 4));
						break;
						case "bool":
						fs.Write(BitConverter.GetBytes(bool.Parse(row[j].ToString()), 0, 4));
						break;
						case "string":
						bytes = Encoding.UTF8.GetBytes(row[j].ToString());
						fs.Write(BitConverter.GetBytes(bytes.Length), 0, 4);
						fs.Write(bytes, 0, bytes.Length);
						break;
					}
				}
			}
			fs.Close();
		}
	}
```

### Excel数据文件的使用
![[BinaryDataMgr.cs]]