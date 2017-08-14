title: Windows时间操作
---
&emsp;本文主要总结了Windows下的主要的时间操作

# 1.CTime

CTime的使用相对简单，这里把MSDN上的文档直接copy过来：

Name|Description
----|-----------
CTime::Format|Converts a CTime object into a formatted string — based on the local time zone.
CTime::FormatGmt|Converts a CTime object into a formatted string — based on UTC.
CTime::GetAsDBTIMESTAMP|Converts the time information stored in the CTime object to a Win32-compatible DBTimeStamp structure.
CTime::GetAsSystemTime|Converts the time information stored in the CTime object to a Win32-compatible SYSTEMTIME structure.
CTime::GetCurrentTime|Creates a CTime object that represents the current time (static member function).
CTime::GetDay|Returns the day represent by the CTime object.
CTime::GetDayOfWeek|Returns the day of the week represented by the CTime object.
CTime::GetGmtTm|Breaks down a CTime object into components — based on UTC.
CTime::GetHour|Returns the hour represented by the CTime object.
CTime::GetLocalTm|Breaks down a CTime object into components — based on the local time zone.
CTime::GetMinute|Returns the minute represented by the CTime object.
CTime::GetMonth|Returns the month represented by the CTime object.
CTime::GetSecond|Returns the second represented by the CTime object.
CTime::GetTime|Returns a __time64_t value for the given CTime object.
CTime::GetYear|Returns the year represented by the CTime object.
CTime::Serialize64|Serializes data to or from an archive.
 

&emsp;记录一下CTime::Format的参数

- %a    缩写的星期名称
- %A    完整星期名称
- %b    缩写的月份名称
- %B    完整的月份名称。
- %c    日期和时间表示恰当的区域设置
- %d    日期为十进制数字 (01 - 31) 的月份
- %H    以 24 小时格式 (00 - 23) 的点
- %I    以 12 小时格式 (01 - 12) 的点
- %j    日期为十进制数字 (001 - 366)。
- %m    为十进制数字 (01 - 12) 的月份
- %M    为十进制数字 (00 - 59) 的分钟
- %p    12 小时时钟的当前区域设置的 A.M/P.M. 指示器
- %S    其次为十进制数字 (00 - 59)
- %U    周为十进制数字的年份，周日为一周的第一天 (00 - 53)
- %w    周为十进制数字 (0 - 6 ；0 是星期天)
- %W    周为十进制数字的年份，星期一为一周的第一天 (00 - 53)
- %x    当前区域设置的日期显示
- %X    当前区域设置的时间显示
- %y    无世纪年，为十进制数字 (00 - 99)
- %Y    世纪年，为十进制数字
- %z, %Z    根据注册表设置，无论是时区名称或时区缩写，如果时区未知，则没有字符
- %%    百分号

# 2.CTimeSpan

&emsp;同样，对于CTimeSpan,也只是将MSDN的文档copy一下：

Name|Description
----|-----------
CTimeSpan::Format|Converts a CTimeSpan into a formatted string.
CTimeSpan::GetDays|Returns a value that represents the number of complete days in this CTimeSpan.
CTimeSpan::GetHours|Returns a value that represents the number of hours in the current day (–23 through 23).
CTimeSpan::GetMinutes|Returns a value that represents the number of minutes in the current hour (–59 through 59).
CTimeSpan::GetSeconds|Returns a value that represents the number of seconds in the current minute (–59 through 59).
CTimeSpan::GetTimeSpan|Returns the value of the CTimeSpan object.
CTimeSpan::GetTotalHours|Returns a value that represents the total number of complete hours in this CTimeSpan.
CTimeSpan::GetTotalMinutes|Returns a value that represents the total number of complete minutes in this CTimeSpan.
CTimeSpan::GetTotalSeconds|Returns a value that represents the total number of complete seconds in this CTimeSpan.
CTimeSpan::Serialize64|Serializes data to or from an archive.

&emsp;记录一下CTimeSpan::Format的参数：

- %D   Total days in this CTimeSpan
- %H   Hours in the current day
- %M   Minutes in the current hour
- %S   Seconds in the current minute
- %%   Percent sign

# 3.__time64_t和time_t

&emsp;定义如下：
``` C++

    typedef __time64_t time_t;  /* time value */
    typedef __int64 __time64_t; /* 64-bit time value */

```

``` C++

    time_t m_now;
    time(&m_now); // 获取当前时间

```

# 4.mstruct tm

``` C++

    struct tm {
    	int tm_sec; /* seconds after the minute - [0,59] */
    	int tm_min; /* minutes after the hour - [0,59] */
    	int tm_hour;/* hours since midnight - [0,23] */
    	int tm_mday;/* day of the month - [1,31] */
    	int tm_mon; /* months since January - [0,11] */
    	int tm_year;/* years since 1900 */
    	int tm_wday;/* days since Sunday - [0,6] */
    	int tm_yday;/* days since January 1 - [0,365] */
    	int tm_isdst;   /* daylight savings time flag 夏令时标识符，实行夏令时的时候，tm_isdst为正。 */
    	/* 不实行夏令时的进候，tm_isdst为0；不了解情况时，tm_isdst()为负*/
    };

```
# 4.1 CTime与tm

``` C++

    CTime time(CTime::GetCurrentTime());
    tm t1, t2;
    time.GetLocalTm(&t1);
    time.GetGmtTm(&t2);//返回的UTC时间

```



``` C++

    struct tm tm1;
    time_t tt2 = mktime(&tm1);

```