---
layout: post
title: Multi-Stage MSBuild AppLocker Bypass
published: false
---
![msbuild]({{ site.url }}/images/msbuild-csproj.png)

### With regular external vulnerability scans and (hopefully) pentration tests being undertaken, the perimeter is typically not the easy route into an organisation.

People click, and despite simulated phishing campaigns and training there can never be complete assurance against this.  So it is critically important that organisations implement a multi-layered "defence in depth" strategy. Application whitelisting is an effective means of ensuring that only _known good_ applications can run.

![rules]({{ site.url }}/images/applocker-default-rules.png)

