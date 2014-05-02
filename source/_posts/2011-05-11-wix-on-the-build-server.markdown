---
layout: post
title: "WiX on the build server"
date: 2011-05-11 23:50:00 +0930
comments: true
categories: [WiX, c#]
---

First post in a long while and its about WiX. I was trying to get a WiX project to build without having to install it on the build server. First off I followed this [Integrating WiX Projects into Daily Builds](http://wix.sourceforge.net/manual-wix3/daily_builds.htm). I had to conditionally set the $(SourceCodeControlRoot) like so

{% codeblock lang:xml %}
<PropertyGroup>
  <SourcesDirectory Condition=" $(SourcesDirectory) == ''">$(MSBuildProjectDirectory)\..\</SourcesDirectory>
  <WixToolPath>$(SourcesDirectory)tools\wix\3.5\</WixToolPath>
  <WixTargetsPath>$(WixToolPath)Wix.targets</WixTargetsPath>
  <WixTasksPath>$(WixToolPath)wixtasks.dll</WixTasksPath>
</PropertyGroup>
{% endcodeblock %}

However when building the project it crashed visual studio with a help error message like 

{% blockquote %}"could not load Candle.exe or one of its dependencies, Operation Not Supported".{% endblockquote %} 

Googling turned up a bunch of issues around 64bit image formats which didn't apply. Digging a little deeper revealed this error message 

{% blockquote %}"An attempt was made to load an assembly from a network location which would have caused the assembly to be sandboxed in previous versions of the .NET Framework. This release of the .NET Framework does not enable CAS policy by default, so this load may be dangerous. If this load is not intended to sandbox the assembly, please enable the loadFromRemoteSources switch. See http://go.microsoft.com/fwlink/?LinkId=155569 for more information."{% endblockquote%}

I wasn't loading it from a network location it was right there on the local file system. I had downloaded the latest binaries zip file for WiX 3.5 and unzipped it straight into the directory like the above link mentions. Since it was download from the interwebs the files had been marked as blocked.

Now what I needed was a way to remove the blocked attribute from all the files, there was no way I wanted to do it manually. Luckily SysInternals has a tool called [Streams](http://technet.microsoft.com/en-us/sysinternals/bb897440) which will clear that attribute from all files in a directory. 

How I got those files into that state in the first place I'm not sure because I tried it again and the files weren't blocked. At least I managed to figure it out.