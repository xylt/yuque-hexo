---
title: Jsoncpp使用说明
date: '2024-03-08 09:52:02'
updated: '2024-08-18 22:54:44'
---
JSON全称为JavaScript ObjectNotation，它是一种轻量级的数据交换格式，易于阅读、编写、解析。jsoncpp是c++解析JSON串常用的解析库之一。
jsoncpp的配置下载[jsoncpp的github](https://github.com/open-source-parsers/jsoncpp)
### 添加到工程
#### 方法一：使用Jsoncpp包中的.cpp和.h文件
解压上面下载的jsoncpp-master.zip文件，把jsoncpp-master\include\json文件夹和jsoncpp-master\src\lib_json文件夹里的全部文件拷贝到工程目录下，并且添加到到VS工程中。 在需要使用JsonCpp的文件中包含json头文件即可，如：#include “json/json.h”。另外，需要将json_reader.cpp、json_value.cpp和json_writer.cpp三个文件的Precompiled Header属性设置为Not Using Precompiled Headers，否则编译会出现错误。
#### 方法二：使用Jsoncpp生成的lib文件
解压上面下载的jsoncpp-master.zip文件，在jsoncpp-master\makefiles\vs71目录里找到jsoncpp.sln，用VS编译，默认生成静态链接库。 在工程中引用，只需要包含include/json下的头文件及生成的.lib文件即可。 在需要使用JsonCpp的文件中添加#pragma comment(lib.”json_vc71_libmt.lib”)，在工程属性中Linker下Input中Additional Dependencies写入lib文件名字（Release下为json_vc71_libmt.lib，Debug为json_vc71_libmtd.lib）注意：Jsoncpp的lib工程编译选项要和VS工程中的编译选项保持一致。如lib文件工程编译选项为MT（或MTd），VS工程中也要选择MT（或MTd），否则会出现编译错误问题。
### json文件

```
[
   {
      "ColWidth" : [ 33, 63, 53, 60, 63, 66, 47, 63, 57, 65, 65, 64, 66, 85 ],
      "DlgName" : "窗口1",
      "FirstRowHeight" : 55,
      "RowHeight" : 19
   },
   {
      "ColWidth" : [ 50, 50, 50, 33, 63, 53 ],
      "DlgName" : "窗口2",
      "FirstRowHeight" : 50,
      "RowHeight" : 30
   }
]
```
### 打开json文件

```
void JsonOpre::OpenJsonFile()
{
	CString m_StrFilePath;
	// 设置过滤器   
	TCHAR szFilter[] = _T("json文件(*.json)|*.json|所有文件(*.*)|*.*||");   
	// 构造保存文件对话框   
	CFileDialog fileDlg(TRUE, _T("读取JSON"), NULL, 0, szFilter);  
	// 显示保存文件对话框
	if (IDOK == fileDlg.DoModal())
	{
		// 如果点击了菜单栏上的"另存为"按钮，则选择的保存的文件路径
		m_StrFilePath = fileDlg.GetPathName();
		std::string str = Auxiliary::CStringToString(m_StrFilePath);
		if(ParseJsonFromFile(str.c_str())) 
			acedAlert(_T("数据成功！"));
		else
			acedAlert(_T("数据保存失败！"));
	}
}
```
### 读取json文件

```
bool JsonOpre::ParseJsonFromFile(const string& filename)
{
	// 解析json用Json::Reader  
	Json::Reader reader;  
	// Json::Value是一种很重要的类型，可以代表任意类型。如int, string, object, array...  
	Json::Value root;

	std::ifstream is;  
	CString cfilename = filename.c_str();
	is.open (cfilename, std::ios::binary );    
	if (reader.parse(is, root))  
	{  
		std::string code;  
		if (root.size() == 0)  // 访问节点，Access an object value by name, create a null member if it does not exist.  
			return false;
		// 遍历数组  
		DlgAttrVec.clear();
		for(int i = 0; i < (int)root.size(); i++)  
		{  
			string Dlgname = root[i]["DlgName"].asString();
			int FirstRowHeight = root[i]["FirstRowHeight"].asInt();
			int RowHeight = root[i]["RowHeight"].asInt();
			int ColWidth_size = root[i]["ColWidth"].size();
			vector<int> ColWidthVec;
			for(int j = 0; j < ColWidth_size; j++)
			{
				ColWidthVec.push_back(root[i]["ColWidth"][j].asInt());
			}
			DlgAttrVec.push_back(*(new DlgAttr(Dlgname, FirstRowHeight, RowHeight, ColWidthVec)));
		}  
	}  
	is.close(); 

	return true; 
}
```
### 存储json文件

```
bool JsonOpre::DataSaveJsonInFile(const string& filename)
{
	Json::Value root;
	Json::Value NewDlg; 
	Json::Value arrayColWidth;   // 构建对象 

	root.clear();
	for(int i = 0; i < DlgAttrVec.size(); i++){
		NewDlg["DlgName"] = DlgAttrVec[i].Dlgname;
		NewDlg["FirstRowHeight"] = DlgAttrVec[i].FirstRowHeight;
		NewDlg["RowHeight"] = DlgAttrVec[i].RowHeight;
		int ColWidth_size = (int)DlgAttrVec[i].ColWidthVec.size();
		arrayColWidth.clear();
		for(int j = 0; j < ColWidth_size; j++) {
				arrayColWidth.append(DlgAttrVec[i].ColWidthVec[j]);
		}
		NewDlg["ColWidth"] = arrayColWidth;
		root.append(NewDlg);  // 插入数组成员 
	}
	//输出（另起一个方法）
	//转换为字符串（带格式）
	std::string out = root.toStyledString();

	std::ofstream os; 
	CString cfilename = filename.c_str();
	os.open (cfilename, std::ios::binary );    
	os<<out;
	os.close();   
	return true;
}
```

 
