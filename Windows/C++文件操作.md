---
title: C++中的文件操作
---

主要总结了以下几种类型的相关操作：

- **文件读写**
- **文件夹及文件的操作**
- **文件打开以及保存对话框**

>MSDN搜索关键字：fstream.

 [TOC]

## 一、C++文件流操作
&ensp;C++提供的文件流ifstream、ofsteam和fstream进行文件操作，文件流操作都是基于流的操作，分别继承于IO类的istream、ostream、stream。由于该文件流是继承于IO类,所以IO类的基本方法都是可以使用的。


### 1.1 打开文件

&emsp;在文件流中，有一个成员函数open()，就是用来打开文件的（有open有相同的构造函数），其原型是：

``` C++
	void open(const char* filename,int mode,int access);
```


**参数：**

- filename:
The name of the file to open.

- mode:
One of the enumerations in ios_base::openmode.

- access：
The default file opening protection, equivalent to the shflag parameter in _fsopen, _wfsopen.



**文件打开方式**

The type is a bitmask type that describes an object that can store the opening mode for several iostreams objects. The distinct flag values (elements) are:

- **app**, to seek to the end of a stream before each insertion.
- **ate**, to seek to the end of a stream when its controlling object is first created.
- **binary**, to read a file as a binary stream, rather than as a text stream.
- **in**, to permit extraction from a stream.
- **out**, to permit insertion to a stream.
- **trunc**, to delete contents of an existing file when its controlling object is created.

The argument access is a constant expression consisting of one of the following manifest constants, defined in Share.h.

Term | Definition
---------- | ----------------------
_SH_COMPAT | Sets Compatibility mode for 16-bit applications.
_SH_DENYNO | Permits read and write access.
_SH_DENYRD | Denies read access to the file.
_SH_DENYRW | Denies read and write access to the file.
_SH_DENYWR | Denies write access to the file.

### 1.2 文件关闭

&emsp;打开的文件使用完成后一定要关闭，文件流提供了成员函数close()来完成此操作。

### 1.3 读写文件

&emsp;读写文件分为文本文件和二进制文件的读取，对于文本文件的读取比较简单，用插入器和析取器就可以了；而对于二进制的读取就要复杂些，下要就详细的介绍这两种方式

#### 1.3.1 文本文件的读写 
&emsp;文本文件的读写很简单：用插入器(<<)向文件输出；用析取器(>>)从文件输入。假设file1是以输入方式打开，file2以输出打开。示例如下：

``` C++
	file2<<"I Love You";//向文件写入字符串"I Love You" 
	int i; 
	file1>>i;//从文件输入一个整数值。
```

&emsp;这种方式还有一种简单的格式化能力，比如可以指定输出为16进制等等，具体的格式有以下一些操纵符功能输入/输出：

- dec 格式化为十进制数值数据  输入和输出
- endl 输出一个换行符并刷新此流  输出 
- ends 输出一个空字符  输出 
- flush 刷新缓存区  输出
- hex 格式化为十六进制数值数据  输入和输出 
- oct 格式化为八进制数值数据  输入和输出 
- setpxecision(int p) 设置浮点数的精度位数  输出
比如要把123当作十六进制输出：

``` C++
	file1<<123<<hex;
```
#### 1.3.2 二进制文件的读写

- put() 

&emsp;put()函数向流写入一个字符，其原型是ofstream &put(char ch)，使用也比较简单，如file1.put('c');就是向流写一个字符'c'。

- get() 

&emsp;get()函数比较灵活，有3种常用的重载形式：

- 一种就是和put()对应的形式：ifstream &get(char &ch);功能是从流中读取一个字符，结果保存在引用ch中，如果到文件尾，返回空字符。


- 一种重载形式的原型是： int get();这种形式是从流中返回一个字符，如果到达文件尾，返回EOF。


- 还有一种形式的原型是：ifstream &get(char *buf,int num,char delim='/n')；这种形式把字符读入由 buf 指向的数组，直到读入了 num个字符或遇到了由 delim 指定的字符，如果没使用 delim 这个参数，将使用缺省值换行符'/n'。例如：

``` C++
	file2.get(str1,127,'A');//从文件中读取字符到字符串str1，当遇到字符'A'或读取了127个字符时终止。
```

- 读写数据块


&emsp;要读写二进制数据块，使用成员函数read()和write()成员函数，它们原型如下：

``` C++
	read(unsigned char *buf,int num); 
	write(const unsigned char *buf,int num);
```

&emsp;read()从文件中读取 num 个字符到 buf 指向的缓存中，如果在还未读入 num 个字符时就到了文件尾，可以用成员函数 int gcount();来取得实际读取的字符数；
&emsp;而 write() 从buf 指向的缓存写 num 个字符到文件中，值得注意的是缓存的类型是 unsigned char *，有时可能需要类型转换。例：

``` C++
	unsigned char str1[]="I Love You"; 
	int n[5]; 
	ifstream in("xxx.xxx"); 
	ofstream out("yyy.yyy"); 
	out.write(str1,strlen(str1));//把字符串str1全部写到yyy.yyy中 
	in.read((unsigned char*)n,sizeof(n));//从xxx.xxx中读取指定个整数，注意类型转换 
	in.close();
	out.close();
```
#### 1.4 检测EOF

&emsp;成员函数eof()用来检测是否到达文件尾，如果到达文件尾返回非0值，否则返回0。原型是:

``` C++
	int eof();
```

#### 1.5 文件定位
&emsp;和C的文件操作方式不同的是，C++ I/O系统管理两个与一个文件相联系的指针。一个是读指针，它说明输入操作在文件中的位置；另一个是写指针，它下次写操作的位置。每次执行输入或输出时，相应的指针自动变化。所以，C++的文件定位分为读位置和写位置的定位，对应的成员函数是 seekg()和 seekp()，seekg()是设置读位置，seekp是设置写位置。它们最通用的形式如下：
``` C++
    istream &seekg(streamoff offset,seek_dir origin);
    ostream &seekp(streamoff offset,seek_dir origin);
```
&emsp;streamoff定义于 iostream.h 中，定义有偏移量 offset 所能取得的最大值，seek_dir 表示移动的基准位置，是一个有以下值的枚举：

``` C++
	ios::beg：　　文件开头
	ios::cur：　　文件当前位置
	ios::end：　　文件结尾
```

&emsp;这两个函数一般用于二进制文件，因为文本文件会因为系统对字符的解释而可能与预想的值不同。
例：

``` C++
    file1.seekg(1234,ios::cur);//把文件的读指针从当前位置向后移1234个字节
    file2.seekp(1234,ios::beg);//把文件的写指针从文件开头向后移1234个字节
```