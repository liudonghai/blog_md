---
title: MFC文件操作
---
 
本文记录了MFC中常见的文件相关操作


 
# 1．文件的打开/保存对话框
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

&emsp;对于MFC提供的这个类使用相对简单，只需要根据你需要的风格选择相应的dwFlags值即可，这里有两点值得注意的是：一是lpszFilter的写法，**必须以"|"结尾**，再一个就是当选择多个文件的时候如何去存储相应的数据，下面显示一个选择多个文件的简单例子：

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

# 2．CFile类的文件操作
&emsp;文件的读写非常重要，下面将重点进行介绍。文件读写的最普通的方法是直接使用CFile进行，如文件的读写可以使用下面的方法：

``` C++

	INT i;
	CString	cstr;
	ULONGLONG ulPosition = 0;
	ULONGLONG ulLength;
	CHAR cWriteBuffer1[20];
	CHAR cWriteBuffer2[20];
	CHAR cReadBuffer[21];
	memset(cWriteBuffer1, L'a', 20);
	memset(cWriteBuffer2, L'b', 20);
	memset(cReadBuffer, '0', 21);
	
	CFile m_File(_T("text.txt"), CFile::modeReadWrite|CFile::modeNoTruncate);
	m_File.Write(cWriteBuffer1, sizeof(cWriteBuffer1));
	m_File.Write(cWriteBuffer2, sizeof(cWriteBuffer2));
	m_File.Flush();
	m_File.SeekToBegin();
	ulLength = m_File.GetLength();
	
	while (ulPosition < ulLength)
	{
		i = m_File.Read(cReadBuffer, sizeof(cReadBuffer)-1);
		cReadBuffer[i] = '\0';
		cstr=cReadBuffer;
		MessageBox(cstr);
		ulPosition += i;
	}

```
CFile类是MFC文件操作的类的基类，还派生的有CStdioFile、CMemFile、CSharedFile（派生于CMemFile），CFile类有常用的一些文件操作的方法，如下：

Name|Description
----|--------- 
CFile::Abort|Closes a file ignoring all warnings and errors.
CFile::Close|Closes a file and deletes the object.
CFile::Duplicate|Constructs a duplicate object based on this file.
CFile::Flush|Flushes any data yet to be written.
CFile::GetFileName|Retrieves the filename of the selected file.
CFile::GetFilePath|Retrieves the full file path of the selected file.
CFile::GetFileTitle|Retrieves the title of the selected file.
CFile::GetLength|Retrieves the length of the file.
CFile::GetPosition|Retrieves the current file pointer.
CFile::GetStatus|Retrieves the status of the open file, or in the static version, retrieves the status of the specified file (static, virtual function).
CFile::LockRange|Locks a range of bytes in a file.
CFile::Open|Safely opens a file with an error-testing option.
CFile::Read|Reads (unbuffered) data from a file at the current file position 
CFile::Remove|Deletes the specified file (static function) 
CFile::Rename|Renames the specified file (static function).
CFile::Seek|Positions the current file pointer. 
CFile::SeekToBegin|Positions the current file pointer at the beginning of the file. 
CFile::SeekToEnd|Positions the current file pointer at the end of the file.
CFile::SetFilePath|Sets the full file path of the selected file.
CFile::SetLength|Changes the length of the file.
CFile::SetStatus|Sets the status of the specified file (static, virtual function).
CFile::UnlockRange|Unlocks a range of bytes in a file.
CFile::Write|Writes (unbuffered) data in a file to the current file position.
 
&emsp;CFile提供的方法最为基础，但是使用并不是那么方便。

# 3.CArchive使用
　
``` C++

	CFile 	m_File;
	CString cstrTemp1("StringTmp11");
	CString cstrTemp2("StringTmp22");
	CString cstrTemp;
	CFile::Remove(L"TEST.txt");
	m_File.Open(L"TEST.txt", CFile::modeCreate | CFile::modeNoTruncate | CFile::modeReadWrite);
	CArchive m_ArchiveStore(&m_File, CArchive::store);
	m_ArchiveStore.WriteString(cstrTemp1);
	m_ArchiveStore.WriteString(L"\n");
	m_ArchiveStore.WriteString(cstrTemp2);
	m_ArchiveStore.WriteString(L"\n");
	m_ArchiveStore.Close();

	//对文件进行读操作
	m_File.SeekToBegin();
	CArchive m_ArchiveLoad(&m_File, CArchive::load);
	m_ArchiveLoad.ReadString(cstrTemp);
	MessageBox(cstrTemp);
	m_ArchiveLoad.ReadString(cstrTemp);
	MessageBox(cstrTemp);
	m_ArchiveLoad.Close();
	m_File.Close();

```


>**注意：**

> - 该类还有类似于流操作的<<和>>操作符，但是我在使用的时候发现老是会出问题，不如直接使用write好用。
> - CArchive构造函数内的打开模式不能进行多选，即不能进行或操作。


<br>
&emsp;CArchive还有进行类以及对象的读写,下面是MSDN上例子：

``` C++

    CFile myFile(_T("My__test__file.dat"), 
       CFile::modeCreate | CFile::modeReadWrite);
    
    // Create a storing archive.
    CArchive arStore(&myFile, CArchive::store);
    
    // Store the class CAge in the archive.
    arStore.WriteClass(RUNTIME_CLASS(CAge));
    
    // Close the storing archive.
    arStore.Close();
    
    // Create a loading archive.
    myFile.SeekToBegin();
    CArchive arLoad(&myFile, CArchive::load);
    
    // Load a class from the archive.
    CRuntimeClass* pClass = arLoad.ReadClass();
    if (!pClass->IsDerivedFrom(RUNTIME_CLASS(CAge)))
    {
       arLoad.Abort();
    }

```

``` C++

    CFile myFile(_T("My__test__file.dat"), 
       CFile::modeCreate | CFile::modeReadWrite);
    CAge age(21), *pAge;
    
    // Create a storing archive.
    CArchive arStore(&myFile, CArchive::store);
    
    // Write the object to the archive
    arStore.WriteObject(&age);
    
    // Close the storing archive
    arStore.Close();
    
    // Create a loading archive.
    myFile.SeekToBegin();
    CArchive arLoad(&myFile, CArchive::load);
    
    // Verify the object is in the archive.
    pAge = (CAge*) arLoad.ReadObject(RUNTIME_CLASS(CAge));
    ASSERT(age == *pAge);

```

# 4.CStdioFile文件操作

&emsp;如果你要进行的文件操作只是简单的读写整行的字符串，还可以使用CStdioFile，用它来进行此类操作非常方便.

&emsp;该类的方法：

Name|Description
----|----------
CStdioFile::Open|Overloaded. Open is designed for use with the default CStdioFile constructor (Overrides CFile::Open).
CStdioFile::ReadString|Reads a single line of text.
CStdioFile::Seek|Positions the current file pointer.
CStdioFile::WriteString|Writes a single line of text.
 

``` C++

　　CFile::Remove(L"Test.txt");
	CStdioFile m_StdioFile(L"Test.txt", CFile::modeCreate | CFile::modeNoTruncate | CFile::modeReadWrite| CFile::typeBinary);
	CString cstrRead;
	CString cstrWrite(L"this is a test file.\n");
	for (INT i = 0; i < 10; i++)
	{
		m_StdioFile.WriteString(cstrWrite);
	}
	m_StdioFile.SeekToBegin();
	while (m_StdioFile.GetPosition() < m_StdioFile.GetLength())
	{
		m_StdioFile.ReadString(cstrRead);
		MessageBox(cstrRead);
	}
	m_StdioFile.Close();

```
&emsp;如果想追加到原有txt文件尾，那么需要在写入之前调用SeekToEnd.像上述的写入方式将覆盖原有的数据。

# 5．临时文件的使用

&emsp;临时文件的使用方法基本与常规文件一样，只是文件名应该调用函数GetTempFileName()获得。它的第一个参数是建立此临时文件的路径，第二个参数是建立临时文件名的前缀，第三个参数为0时有系统创建改文件，非0时有手动创建，但是手动创建时不保证不重名，第四个参数用于得到建立的临时文件名。得到此临时文件名以后，你就可以用它来建立并操作文件了，如：

``` C++

    char szTempPath[_MAX_PATH],szTempfile[_MAX_PATH];
    GetTempPath(_MAX_PATH, szTempPath);
    GetTempFileName(szTempPath,_T ("my_"),0,szTempfile);
    CFile m_tempFile(szTempfile,CFile:: modeCreate|CFile:: modeWrite);
    m_tempFile.Close();

```

# 6.文件的查找

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
    			file_find(cstrPath，cstrKey，bSubDirectory);
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

# 7．文件的复制、删除
&emsp;MFC中没有提供直接进行这些操作的功能，因而要使用SDK。SDK中的文件相关函数常用的有CopyFile()、CreateDirectory()、DeleteFile()、MoveFile()。

## 7.1 判断文件、文件夹是否存在

&emsp;判断文件是否存在：_access

``` C++

     View ColorizedCopy to Clipboardint _access( 
       const char *path, 
       int mode 
    );
    int _waccess( 
       const wchar_t *path, 
       int mode 
    );
```

Return value:

&emsp;Each function returns 0 if the file has the given mode. The function returns –1 if the named file does not exist or does not have the given mode; in this case, errno is set as shown in the following table.


- EACCES:Access denied: the file's permission setting does not allow specified access.
- ENOENT:File name or path not found.
- EINVAL:Invalid parameter.

mode:

mode value|Checks file for
----------|---------------
00|Existence only
02|Write-only
04|Read-only
06|Read and write

&emsp;判断文件夹是否存在：PathIsDirectory

 
## 7.2 文件夹创建、文件删除、文件移动、文件复制

- CreateDirectory(FileName,securityAttributes);
- DeleteFile(FileName);
- MoveFile(ExistingFile,newFile);
- CopyFile(ExistingFile,NewFile,Exist);
 
>CopyFile: If this parameter is TRUE and the new file specified by lpNewFileName already exists, the function fails. If this parameter is FALSE and the new file already exists, the function overwrites the existing file and succeeds.

## 7.3文件创建

