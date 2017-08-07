---
title: MFC文件操作
---
 
本文记录了MFC中常见的文件相关操作

# 1.文件的查找

&emsp;CFileFind是为另两类查找特殊服务器设计的MFC类的基类，CGopherFileFind在Gopher服务器上工作，CFtpFileFind在FTP服务器上工作，这些类为用户查找文件提供了一种无缝机制，与服务器协议、文件类型、地点、本地机器或远程服务器无关。

``` C++
	CString strFileTitle;
	CFileFind finder;
	BOOL bWorking = finder.FindFile("C:\windows\sysbkup\*.cab");
	if(bWorking)
	{
		bWorking=finder.FindNextFile();
		strFileTitle=finder.GetFileTitle();
	}

```

&emsp;简单介绍一下该类的几个常用函数：

Function | Describe
----------- | ----------
CFileFind::FindFile | Searches a directory for a specified file name.
CFileFind::FindNextFile | Continues a file search from a previous call to FindFile.
CFileFind::IsDirectory | Determines if the found file is a directory.

&emsp;还有部分获取各类文件属性的方法，MSDN搜索CFileFind就可以找到了。**值得注意的是包括IsDirectory在内的几个方法，在调用的时候，必须先调用FindNextFile，否则程序就会报错**。个人理解FindFile这个方法只能让你找到这个路径指示的文件或者文件夹，但是并不能获取到相关的数据。

下面给了两个小代码，一个是判断文件或者是目录是否存在的方法,另一个是递归搜索文件的方法：

``` C++
/*
函数功能：判断给定路径的文件或者文件夹是否存在
参数：
	输入：
	cstrPath:文件路径
返回值：
	不存在：0;
	存在且是文件：1；
	存在且是目录：2.
*/
INT file_check(const CString cstrPath)
{
	assert(!cstrPath.IsEmpty());
	INT iResult = 0;
	BOOL bResult;
	CFileFind m_FileFind;

	bResult = m_FileFind.FindFile(cstrPath);
	if (bResult)
	{
		m_FileFind.FindNextFileW();
		if (m_FileFind.IsDirectory())
		{
			iResult = 2;
		}
		else
		{
			iResult = 1;
		}
	}
	else
	{
		iResult = 0;
	}
	m_FileFind.Close();
	return iResult;
}
```
 
``` C++
/*
功能：递归查找指定路径下，包含特定关键字的文件。
参数：
	输出：
	cstrDirPath：文件路径；
	pKeyName：关键字；
	bSubDirectory：是否查找子文件夹，TRUE表示需要。
*/
VOID CTBSScriptManager::file_find(const CString cstrDirPath, const CString cstrKey = L"*.*", 
				bool bSubDirectory = TRUE)
{
	assert(!cstrDirPath.IsEmpty());

	CFileFind m_FileFind;
	BOOL bResult;
	
	bResult=m_FileFind.FindFile(cstrDirPath+L"\\*.*");
	while(bResult)
	{
		bResult = m_FileFind.FindNextFile();
		if (m_FileFind.IsDots())
			continue;
		if (m_FileFind.IsDirectory() && bSubDirectory)
		{
			CString cstrPath = m_FileFind.GetFilePath();
			file_find(cstrPath);
		}
		else
		{
			CString cstrPath = m_FileFind.GetFilePath();
			CString cstrName = m_FileFind.GetFileName();
			BSTR bstr;
			bstr = cstrName.AllocSysString();
			wregex m_wregex(cstrKey);
			if(regex_search(bstr, m_wregex))
			{
				//TODO:	adding execute by yourself when the file is found.
			}
		}
	}
	m_FileFind.Close();
}

```
 
# 2．文件的打开/保存对话框
&emsp;让用户选择文件进行打开和存储操作时，就要用到文件打开/保存对话框。MFC的类CFileDialog用于实现这种功能。使用CFileDialog声明一个对象时，第一个BOOL型参数用于指定文件的打开或保存，当为TRUE时将构造一个文件打开对话框，为FALSE时构造一个文件保存对话框。
&emsp;在构造CFileDialog对象时，如果在参数中指定了OFN_ALLOWMULTISELECT风格，则在此对话框中可以进行多选操作。此时要重点注意为此CFileDialog对象的m_ofn.lpstrFile分配一块内存，用于存储多选操作所返回的所有文件路径名，如果不进行分配或分配的内存过小就会导致操作失败。下面这段程序演示了文件打开对话框的使用方法。


``` C++

explicit CFileDialog(
   BOOL bOpenFileDialog, //文件打开对话框类型，TRUE为打开对话框
   LPCTSTR lpszDefExt = NULL, //默认的文件扩展名
   LPCTSTR lpszFileName = NULL, //显示文件名框的初始文件名
   DWORD dwFlags = OFN_HIDEREADONLY | OFN_OVERWRITEPROMPT,//可以使用自定义对话框一个或多个标记的组合
   LPCTSTR lpszFilter = NULL, //一系列字符串名称的筛选器可应用于文件
   CWnd* pParentWnd = NULL, //指针到文件对话框的父级或所有者窗口
   DWORD dwSize = 0,//OPENFILENAME 结构的大小
   BOOL bVistaStyle = TRUE//指定文件对话框的样式的参数
);

```
对于MFC提供的这个类使用相对简单，只需要根据你需要的风格选择相应的dwFlags值即可，这里有两点值得注意的是：一是lpszFilter的写法，必须以"|"结尾，再一个就是当选择多个文件的时候如何去存储相应的数据，下面显示一个选择多个文件的简单例子：

``` C++

	CString cstrFileBuffer;
	CString cstrFileName;
	POSITION m_Pos;
	CString cstrFile = _T("All file(*.*)|*.*|")\
		_T("*.cpp;*.h|*.cpp;*.h|");

	CFileDialog m_FileDlg(TRUE, NULL, NULL, OFN_HIDEREADONLY | OFN_OVERWRITEPROMPT | OFN_ALLOWMULTISELECT, cstrFile);
	m_FileDlg.m_ofn.lpstrFile = cstrFileBuffer.GetBuffer(1024 * 10);
	cstrFileBuffer.ReleaseBuffer();
	if (IDOK == m_FileDlg.DoModal())
	{
		m_Pos = m_FileDlg.GetStartPosition();
		while (m_Pos != NULL)
		{
			cstrFileName = m_FileDlg.GetNextPathName(m_Pos);
			MessageBox(cstrFileName);
		}
	}
```

# 3．文件的读写
&emsp;文件的读写非常重要，下面将重点进行介绍。文件读写的最普通的方法是直接使用CFile进行，如文件的读写可以使用下面的方法：

``` C++

	//对文件进行读操作
	char sRead[2];
	CFile mFile(_T("user.txt"),CFile::modeRead);
	if(mFile.GetLength()<2)
	return;
	mFile.Read(sRead,2);
	mFile.Close();
	//对文件进行写操作
	CFile mFile(_T("user.txt "), CFile::modeWrite|CFile::modeCreate);
	mFile.Write(sRead,2);
	mFile.Flush();
	mFile.Close();

```
　　虽然这种方法最为基本，但是它的使用繁琐，而且功能非常简单。我向你推荐的是使用CArchive，它的使用方法简单且功能十分强大。首先还是用CFile声明一个对象，然后用这个对象的指针做参数声明一个CArchive对象，你就可以非常方便地存储各种复杂的数据类型了。它的使用方法见下例。
　　
//对文件进行写操作
　　CString strTemp;
　　CFile mFile;
　　mFile.Open("d:\dd\try.TRY",CFile::modeCreate|CFile::modeNoTruncate|CFile::modeWrite);
　　CArchive ar(&mFile,CArchive::store);
　　ar<<　　ar.Close();
　　mFile.Close();
　　//对文件进行读操作
　　CFile mFile;
　　if(mFile.Open("d:\dd\try.TRY",CFile::modeRead)==0)
　　return;
　　CArchive ar(&mFile,CArchive::load);
　 　ar>>strTemp;
	ar.Close();
　　mFile.Close();

　　CArchive的 << 和>> 操作符用于简单数据类型的读写，对于CObject派生类的对象的存取要使用ReadObject()和WriteObject()。使用CArchive的ReadClass()和WriteClass()还可以进行类的读写，如：
　　//存储CAboutDlg类
　　ar.WriteClass(RUNTIME_CLASS(CAboutDlg));
　　//读取CAboutDlg类
　　CRuntimeClass* mRunClass=ar.ReadClass();
　　//使用CAboutDlg类
　　CObject* pObject=mRunClass->CreateObject();
   ((CDialog* )pObject)->DoModal();
　　虽然VC提供的文档/视结构中的文档也可进行这些操作，但是不容易理解、使用和管理，因此虽然很多VC入门的书上花费大量篇幅讲述文档/视结构，但我建议你最好不要使用它的文档。关于如何进行文档/视的分离有很多书介绍，包括非常著名的《Visual C++ 技术内幕》。
　　如果你要进行的文件操作只是简单的读写整行的字符串，我建议你使用CStdioFile，用它来进行此类操作非常方便，如下例。
　　CStdioFile mFile;
　　CFileException mExcept;
　　mFile.Open( "d:\temp\aa.bat", CFile::modeWrite, &mExcept);
　　CString string="I am a string.";
　　mFile.WriteString(string);
　　mFile.Close();


追加到原有txt文件尾

    mFile.Open(temptxt,CFile::modeCreate|CFile::modeWrite|CFile::modeNoTruncate);
 if(mFile==NULL)
  return false;
 mFile.SeekToEnd();
 mFile.WriteString(s);

 file.Close();
　4．临时文件的使用

　　正规软件经常用到临时文件，你经常可以会看到C:WindowsTemp目录下有大量的扩展名为tmp的文件，这些就是程序运行是建立的临时文件。临时文件的使用方法基本与常规文件一样，只是文件名应该调用函数GetTempFileName()获得。它的第一个参数是建立此临时文件的路径，第二个参数是建立临时文件名的前缀，第四个参数用于得到建立的临时文件名。得到此临时文件名以后，你就可以用它来建立并操作文件了，如：
　　char szTempPath[_MAX_PATH],szTempfile[_MAX_PATH];
　　GetTempPath(_MAX_PATH, szTempPath);
　　GetTempFileName(szTempPath,_T ("my_"),0,szTempfile);
　　CFile m_tempFile(szTempfile,CFile:: modeCreate|CFile:: modeWrite);
　　char m_char='a';
　　m_tempFile.Write(&m_char,2);
　　m_tempFile.Close();
　　5．文件的复制、删除等
　　MFC中没有提供直接进行这些操作的功能，因而要使用SDK。SDK中的文件相关函数常用的有CopyFile()、CreateDirectory()、DeleteFile()、MoveFile()。它们的用法很简单，可参考MSDN。

1,判断文件是否存在
    access(filename,mode);
2,对于不同用途又不同的文件操作,其中API函数CreateFile()也是比较有用处理方式,对于巨型文件很合适的其他的楼上的大都说了,不重复了.

[1]显示对话框，取得文件名

CString FilePathName;
CFileDialog dlg(TRUE);///TRUE为OPEN对话框，FALSE为S***E AS对话框
if (dlg.DoModal() == IDOK)
    FilePathName=dlg.GetPathName();

相关信息：CFileDialog 用于取文件名的几个成员函数：
假如选择的文件是C:WINDOWSTEST.EXE
则(1)GetPathName();取文件名全称，包括完整路径。取回C:WINDOWSTEST.EXE
(2)GetFileTitle();取文件全名：TEST.EXE
(3)GetFileName();取回TEST
(4)GetFileExt();取扩展名EXE

[2]打开文件
CFile file("C:HELLO.TXT",CFile::modeRead);//只读方式打开
//CFile::modeRead可改为 CFile::modeWrite(只写),
//CFile::modeReadWrite(读写),CFile::modeCreate(新建)
例子：
{
CFile file;
file.Open("C:HELLO.TXT",CFile::modeCreate|Cfile::modeWrite);
.
.
.
}

[3]移动文件指针
file.Seek(100,CFile::begin);///从文件头开始往下移动100字节
file.Seek(-50,CFile::end);///从文件末尾往上移动50字节
file.Seek(-30,CFile::current);///从当前位置往上移动30字节
file.SeekToBegin();///移到文件头
file.SeekToEnd();///移到文件尾

[4]读写文件
读文件：
char buffer[1000];
file.Read(buffer,1000);
写文件：
CString string("自强不息");
file.Write(string,8);

[5]关闭文件
file.Close();

　　bWorking=finder.FindNextFile();
　　strFileTitle=finder.GetFileTitle();
　　}

　　2．文件的打开/保存对话框
　　让用户选择文件进行打开和存储操作时，就要用到文件打开/保存对话框。MFC的类CFileDialog用于实现这种功能。使用CFileDialog声明一个对象时，第一个BOOL型参数用于指定文件的打开或保存，当为TRUE时将构造一个文件打开对话框，为FALSE时构造一个文件保存对话框。
　　在构造CFileDialog对象时，如果在参数中指定了OFN_ALLOWMULTISELECT风格，则在此对话框中可以进行多选操作。此时要重点注意为此CFileDialog对象的m_ofn.lpstrFile分配一块内存，用于存储多选操作所返回的所有文件路径名，如果不进行分配或分配的内存过小就会导致操作失败。下面这段程序演示了文件打开对话框的使用方法。

　　CFileDialog mFileDlg(TRUE,NULL,NULL,OFN_HIDEREADONLY|OFN_OVERWRITEPROMPT|OFN_ALLOWMULTISELECT,
　　"All Files (*.*)|*.*||",AfxGetMainWnd());
　　CString str(" ",10000);
　　mFileDlg.m_ofn.lpstrFile=str.GetBuffer(10000);
　　str.ReleaseBuffer();
　　POSITION mPos=mFileDlg.GetStartPosition();
　　CString pathName(" ",128);
　　CFileStatus status;
　　while(mPos!=NULL)
　　{
　　pathName=mFileDlg.GetNextPathName(mPos);
　　CFile::GetStatus( pathName, status );
　　}

　　3．文件的读写
　　文件的读写非常重要，下面将重点进行介绍。文件读写的最普通的方法是直接使用CFile进行，如文件的读写可以使用下面的方法：
　　//对文件进行读操作
　　
char sRead[2];
　　CFile mFile(_T("user.txt"),CFile::modeRead);
　　if(mFile.GetLength()<2)
　　return;
　　mFile.Read(sRead,2);
　　mFile.Close();
　　//对文件进行写操作
　　CFile mFile(_T("user.txt "), CFile::modeWrite|CFile::modeCreate);
　　mFile.Write(sRead,2);
　　mFile.Flush();
　　mFile.Close();

　　虽然这种方法最为基本，但是它的使用繁琐，而且功能非常简单。我向你推荐的是使用CArchive，它的使用方法简单且功能十分强大。首先还是用CFile声明一个对象，然后用这个对象的指针做参数声明一个CArchive对象，你就可以非常方便地存储各种复杂的数据类型了。它的使用方法见下例。
　　
//对文件进行写操作
　　CString strTemp;
　　CFile mFile;
　　mFile.Open("d:\dd\try.TRY",CFile::modeCreate|CFile::modeNoTruncate|CFile::modeWrite);
　　CArchive ar(&mFile,CArchive::store);
　　ar<<　　ar.Close();
　　mFile.Close();
　　//对文件进行读操作
　　CFile mFile;
　　if(mFile.Open("d:\dd\try.TRY",CFile::modeRead)==0)
　　return;
　　CArchive ar(&mFile,CArchive::load);
　 　ar>>strTemp;
    　 ar.Close();
　　mFile.Close();

　　CArchive的 << 和>> 操作符用于简单数据类型的读写，对于CObject派生类的对象的存取要使用ReadObject()和WriteObject()。使用CArchive的ReadClass()和WriteClass()还可以进行类的读写，如：
　　//存储CAboutDlg类
　　ar.WriteClass(RUNTIME_CLASS(CAboutDlg));
　　//读取CAboutDlg类
　　CRuntimeClass* mRunClass=ar.ReadClass();
　　//使用CAboutDlg类
　　CObject* pObject=mRunClass->CreateObject();
    　　((CDialog* )pObject)->DoModal();
　　虽然VC提供的文档/视结构中的文档也可进行这些操作，但是不容易理解、使用和管理，因此虽然很多VC入门的书上花费大量篇幅讲述文档/视结构，但我建议你最好不要使用它的文档。关于如何进行文档/视的分离有很多书介绍，包括非常著名的《Visual C++ 技术内幕》。
　　如果你要进行的文件操作只是简单的读写整行的字符串，我建议你使用CStdioFile，用它来进行此类操作非常方便，如下例。
　　CStdioFile mFile;
　　CFileException mExcept;
　　mFile.Open( "d:\temp\aa.bat", CFile::modeWrite, &mExcept);
　　CString string="I am a string.";
　　mFile.WriteString(string);
　　mFile.Close();


追加到原有txt文件尾

    mFile.Open(temptxt,CFile::modeCreate|CFile::modeWrite|CFile::modeNoTruncate);
 if(mFile==NULL)
  return false;
 mFile.SeekToEnd();
 mFile.WriteString(s);

 file.Close();
　4．临时文件的使用

　　正规软件经常用到临时文件，你经常可以会看到C:WindowsTemp目录下有大量的扩展名为tmp的文件，这些就是程序运行是建立的临时文件。临时文件的使用方法基本与常规文件一样，只是文件名应该调用函数GetTempFileName()获得。它的第一个参数是建立此临时文件的路径，第二个参数是建立临时文件名的前缀，第四个参数用于得到建立的临时文件名。得到此临时文件名以后，你就可以用它来建立并操作文件了，如：
　　char szTempPath[_MAX_PATH],szTempfile[_MAX_PATH];
　　GetTempPath(_MAX_PATH, szTempPath);
　　GetTempFileName(szTempPath,_T ("my_"),0,szTempfile);
　　CFile m_tempFile(szTempfile,CFile:: modeCreate|CFile:: modeWrite);
　　char m_char='a';
　　m_tempFile.Write(&m_char,2);
　　m_tempFile.Close();
　　5．文件的复制、删除等
　　MFC中没有提供直接进行这些操作的功能，因而要使用SDK。SDK中的文件相关函数常用的有CopyFile()、CreateDirectory()、DeleteFile()、MoveFile()。它们的用法很简单，可参考MSDN。

1,判断文件是否存在
    access(filename,mode);
2,对于不同用途又不同的文件操作,其中API函数CreateFile()也是比较有用处理方式,对于巨型文件很合适的其他的楼上的大都说了,不重复了.

[1]显示对话框，取得文件名

CString FilePathName;
CFileDialog dlg(TRUE);///TRUE为OPEN对话框，FALSE为S***E AS对话框
if (dlg.DoModal() == IDOK)
    FilePathName=dlg.GetPathName();

相关信息：CFileDialog 用于取文件名的几个成员函数：
假如选择的文件是C:WINDOWSTEST.EXE
则(1)GetPathName();取文件名全称，包括完整路径。取回C:WINDOWSTEST.EXE
(2)GetFileTitle();取文件全名：TEST.EXE
(3)GetFileName();取回TEST
(4)GetFileExt();取扩展名EXE

[2]打开文件
CFile file("C:HELLO.TXT",CFile::modeRead);//只读方式打开
//CFile::modeRead可改为 CFile::modeWrite(只写),
//CFile::modeReadWrite(读写),CFile::modeCreate(新建)
例子：
{
CFile file;
file.Open("C:HELLO.TXT",CFile::modeCreate|Cfile::modeWrite);
.
.
.
}

[3]移动文件指针
file.Seek(100,CFile::begin);///从文件头开始往下移动100字节
file.Seek(-50,CFile::end);///从文件末尾往上移动50字节
file.Seek(-30,CFile::current);///从当前位置往上移动30字节
file.SeekToBegin();///移到文件头
file.SeekToEnd();///移到文件尾

[4]读写文件
读文件：
char buffer[1000];
file.Read(buffer,1000);
写文件：
CString string("自强不息");
file.Write(string,8);

[5]关闭文件
file.Close();
