---
title: 检查windows系统版本
date: 2023-03-20 23:15:21
categories:
  - Windows
---
在开发应用程序的一些场景中，会需要确认当前程序运行所在的操作系统版本是否支持该程序所需要的功能。Windows也提供的几个相关功能的API供开发者使用。不过由于大版本迭代所产生了一些兼容性问题而导致这些早期的API被逐渐弃用。而Windows也提供了一些新的方法来代替他们完成类似的任务。  <!--more-->

### GetVersion和GetVersionEx

该函数在win8.1后被弃用。其调用结果win8及以前的版本正常运行，而在win8.1及以后的版本中都返回6.2（win8的版本号）。

### VerifyVersionInfo

在win10后被弃用。但其行为在win8.1后已经改变。在win8.1后其结果受清单文件manifest所影响。根据官方文档说明其行为如下：

- 如果应用程序没有manifest，**VerifyVersionInfo**会认为操作系统是win8（6.2）

- 如果应用程序有包含win8.1对应GUID的manifest文件，**VerifyVersionInfo**会认为操作系统是win8.1（6.3）

- 如果应用程序有包含win10对应GUID的manifest文件，VerifyVersionInfo会认为操作系统是win10（10.0）

### Version Helper functions

Windows在弃用一些函数的同时也提供了一些新的API供开发者间接的完成版本检查工作。[Version Helper functions](https://learn.microsoft.com/en-us/windows/win32/sysinfo/version-helper-apis)提供了一些诸如**IsWindows10OrGreater**的系统版本检测函数。

### 注册表

Windows注册表中也存放着详细的系统版本信息，可以通过查询注册表来获得想要的版本信息。注册表路径为`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion`

### API

虽然很多API被遗弃，但在ntdll当中还包含着几个可用的API来直接获取操作系统版本。**RtlGetNtVersionNumbers**（未文档化 ），**RtlGetVersion**，**RtlVerifyVersionInfo**。虽然这些函数一般用在驱动程序中，但是在r3也可以正常使用。

使用**GetProcAddress**获取即可。需要注意，未文档化的API不能保证在未来能够持续正常工作。

```cpp
#include <Windows.h>
#include <iostream>
using namespace std;

typedef VOID(*pfRTLGETNTVERSIONNUMBERS)(DWORD*, DWORD*, DWORD*);
typedef NTSYSAPI NTSTATUS(*pfRTLGETVERSION) (PRTL_OSVERSIONINFOW);
typedef NTSYSAPI NTSTATUS(*pfRTLVERIFYVERSIONINFO)(PRTL_OSVERSIONINFOEXW, ULONG, ULONGLONG);

void PrintNtVersion();

int main()
{
	PrintNtVersion();
	return 0;
}

void PrintNtVersion() {
	HMODULE hNtdll = LoadLibrary(L"ntdll.dll");
	if (!hNtdll) {
		cout << "Load ntdll failed" << endl;
		return;
	}
 
 //RtlGetNtVersionNumbers
	pfRTLGETNTVERSIONNUMBERS pfRtlGetNtVersionNumbers = (pfRTLGETNTVERSIONNUMBERS)GetProcAddress(hNtdll, "RtlGetNtVersionNumbers");
	if (!pfRtlGetNtVersionNumbers) {
		cout << "Get proc RtlGetNtVersionNumbers failed" << endl;
		FreeLibrary(hNtdll);
		return;
	}

	DWORD dwMajorVer = 0;
	DWORD dwMinorVer = 0;
	DWORD dwBuildNum = 0;
	pfRtlGetNtVersionNumbers(&dwMajorVer, &dwMinorVer, &dwBuildNum);
	cout << "RtlGetNtVersionNumbers" << endl;
	cout << "Major version:" << dwMajorVer << " Minor version:" << dwMinorVer << " Build number:" << (dwBuildNum & 0xffff) << endl;

 //RtlGetVersion
	pfRTLGETVERSION pfRtlGetVersion = (pfRTLGETVERSION)GetProcAddress(hNtdll, "RtlGetVersion");
	if (!pfRtlGetVersion) {
		cout << "Get proc RtlGetVersion failed" << endl;
		FreeLibrary(hNtdll);
		return;
	}

	RTL_OSVERSIONINFOW VersionInformation = { 0 };
	VersionInformation.dwOSVersionInfoSize = sizeof(RTL_OSVERSIONINFOW);
	NTSTATUS ret = pfRtlGetVersion(&VersionInformation);
	cout << "RtlGetVersion" << endl;
	cout << "Major version:" << VersionInformation.dwMajorVersion << " Minor version:" << VersionInformation.dwMinorVersion << " Build number:" << VersionInformation.dwBuildNumber << endl;

 //RtlVerifyVersionInfo
	pfRTLVERIFYVERSIONINFO pfRtlVerifyVersionInfo = (pfRTLVERIFYVERSIONINFO)GetProcAddress(hNtdll, "RtlVerifyVersionInfo");
	if (!pfRtlVerifyVersionInfo) {
		cout << "Get proc RtlVerifyVersionInfo failed" << endl;
		FreeLibrary(hNtdll);
		return;
	}

	RTL_OSVERSIONINFOEXW VersionInfo = { 0 };
	VersionInfo.dwOSVersionInfoSize = sizeof(RTL_OSVERSIONINFOEXW);
	VersionInfo.dwMajorVersion = 10;
	VersionInfo.dwMinorVersion = 0;
	VersionInfo.dwBuildNumber = 19045;

	ULONG TypeMask = VER_MAJORVERSION | VER_MINORVERSION | VER_BUILDNUMBER;
	ULONGLONG ConditionMask = 0; ;
	VER_SET_CONDITION(ConditionMask, VER_MAJORVERSION, VER_EQUAL);
	VER_SET_CONDITION(ConditionMask, VER_MINORVERSION, VER_EQUAL);
	VER_SET_CONDITION(ConditionMask, VER_BUILDNUMBER, VER_EQUAL);

	NTSTATUS ret_0 = pfRtlVerifyVersionInfo(&VersionInfo, TypeMask, ConditionMask);
	cout << "RtlVerifyVersionInfo" << endl;
	if ((LONG)ret_0 >= 0) {
		cout << "Verify 10.0.19045 success" << endl;
	}
	else {
		cout << "Verify 10.0.19045 false" << endl;

	}


	FreeLibrary(hNtdll);
	return;
}
```

### Windows版本

| Operating system          | Version number            |
| ------------------------- | ------------------------- |
| Windows 11                | 10\.0\*                   |
| Windows 10                | 10\.0\*                   |
| Windows Server 2022       | 10\.0\*                   |
| Windows Server 2019       | 10\.0\*                   |
| Windows Server 2016       | 10\.0\*                   |
| Windows 8.1               | 6\.3\*                    |
| Windows Server 2012 R2    | 6\.3\*                    |
| Windows 8                 | 6\.2                      |
| Windows Server 2012       | 6\.2                      |
| Windows 7                 | 6\.1                      |
| Windows Server 2008 R2    | 6\.1                      |
| Windows Server 2008       | 6\.0                      |
| Windows Vista             | 6\.0                      |
| Windows Server 2003 R2    | 5\.2                      |
| Windows Server 2003       | 5\.2                      |
| Windows XP 64-Bit Edition | 5\.2                      |
| Windows XP                | 5\.1                      |
| Windows 2000              | 5\.0                      |



---



