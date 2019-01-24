---
layout: post
title: SystemPropertiesAdvanced.exe UAC Bypass
published: true
---
![procmon]({{ site.url }}/images/procmon.png){: .center-image }


### Auto-elevating binaries are a good source of bypasses for Windows User Account Control (UAC). "SystemPropertiesAdvanced.exe" and other SystemProperties* binaries can be used to bypass UAC on Windows Server 2019 via a DLL hijacking vulnerability.

The built-in findstr utility can be used to confirm whether the manifest within the binary is set to auto-elevate:

`findstr /C:"<autoElevate>true" C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe`

![auto-elevate]({{ site.url }}/images/auto-elevate.png){: .center-image }

Alternatively, sigcheck can be used to dump the manifest.

`sigcheck.exe -m C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe`

![manifest]({{ site.url }}/images/manifest.png){: .center-image }

After setting the Procmon filter, the auto-elevating SysWOW64 binaries are executed.

![filter]({{ site.url }}/images/filter.png){: .center-image }

This results in a fair amount of output, and so additional Procmon filters to exclude paths starting with "C:\Windows" and "C:\Program " can be applied. Of note, the binary "C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe" attempts to load the DLL "srrstr.dll" from the WindowsApps folder, as it is included in the Windows PATH variable.

![dll]({{ site.url }}/images/dll.png){: .center-image }

![path]({{ site.url }}/images/path.png){: .center-image }

This folder is writeable by unpriviledged users, who may want to install Apps from the Microsoft Store.

A DLL to spawn calc.exe is crafted (tested with DllMain) and saved to the WindowsApps folder. "SystemPropertiesAdvanced.exe" is executed again (from a Medium Integrity Level command prompt), and calc.exe is spawned as a High Integrity Level process.

![uac-bypass]({{ site.url }}/images/uac-bypass.png){: .center-image }

In testing, this DLL hijack / UAC bypass also affects other SysWOW64 SystemProperties* binaries:

```SystemPropertiesAdvanced.exe
SystemPropertiesComputerName.exe
SystemPropertiesHardware.exe
SystemPropertiesProtection.exe
SystemPropertiesRemote.exe
```