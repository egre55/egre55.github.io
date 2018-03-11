---
layout: post
title: Multi-Stage MSBuild AppLocker Bypass
published: false
---
![msbuild]({{ site.url }}/images/msbuild-csproj.png)

### With regular external vulnerability scans and (hopefully) pentration tests being undertaken, the perimeter is typically not the easy route into an organisation.

People click, and despite simulated phishing campaigns and training there can never be complete assurance against credential divulgence or execution of malicious code.  So it is critically important that organisations implement a multi-layered "defence in depth" strategy. Application whitelisting is an important component of such a strategy, and is an effective means of ensuring that only _known good_ applications can run.

AppLocker is a commonly used whitelisting technology and is built into Windows. The default AppLocker rules for Windows 10 are listed below.

![rules]({{ site.url }}/images/applocker-default-rules.png)

Since Powershell has gained recognition for being the attackers language of choice, many organisations are additionally blocking it. However, blocking powershell.exe is often not sufficient and there are many methods by which PowerShell can be instantiated to send a reverse shell reaching out of the organisation to an attacker.

The post will document one such method, with the "misplaced trust binary" MSBuild.exe.



