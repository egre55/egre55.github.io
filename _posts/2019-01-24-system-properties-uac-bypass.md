---
layout: post
title: SystemPropertiesAdvanced.exe DLL Hijacking UAC Bypass
published: true
---
![procmon]({{ site.url }}/images/system_properties_uac_bypass.assets/procmon.png){: .center-image }


### Auto-elevating binaries are a good source of bypasses for Windows User Account Control (UAC). "SystemPropertiesAdvanced.exe" and other SystemProperties* binaries can be used to bypass UAC on Windows Server 2019 via DLL hijacking.

`findstr` can used to examine the auto-elevate behaviour within the embedded manifest:

`findstr /C:"<autoElevate>true" C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe`

![auto-elevate]({{ site.url }}/images/system_properties_uac_bypass.assets/auto-elevate.png){: .center-image }

Alternatively, `sigcheck` can be used to dump the manifest.

`sigcheck.exe -m C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe`

![manifest]({{ site.url }}/images/system_properties_uac_bypass.assets/manifest.png){: .center-image }

After setting the Procmon filter, the auto-elevating SysWOW64 binaries are executed.

![filter]({{ site.url }}/images/system_properties_uac_bypass.assets/filter.png){: .center-image }

This results in a fair amount of output, and so additional Procmon filters to exclude DLL paths starting with "C:\Windows" and "C:\Program Files" can be applied. Of note, the binary "C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe" attempts to load the DLL "srrstr.dll" from the WindowsApps folder. As this folder is included in the PATH variable, Windows will search this location for the required DLL.

![dll]({{ site.url }}/images/system_properties_uac_bypass.assets/dll.png){: .center-image }

![path]({{ site.url }}/images/system_properties_uac_bypass.assets/path.png){: .center-image }

This folder is writeable by unprivileged users, in order to allow installation of Apps from the Microsoft Store.

A DLL to spawn calc.exe is crafted (tested with DllMain) and saved to the WindowsApps folder. "SystemPropertiesAdvanced.exe" is executed again (from a Medium Integrity Level command prompt), and calc.exe is spawned as a High Integrity Level process.

![uac-bypass]({{ site.url }}/images/system_properties_uac_bypass.assets/uac-bypass.png){: .center-image }

In testing, this DLL hijack / UAC bypass also affects other SysWOW64 SystemProperties* binaries:

```SystemPropertiesAdvanced.exe
SystemPropertiesComputerName.exe
SystemPropertiesHardware.exe
SystemPropertiesProtection.exe
SystemPropertiesRemote.exe
```

Setting UAC policy to "Always Notify" mitigates this issue.

**Note:** Microsoft fixed this issue in Windows 10 version 1903 (build 18362).

