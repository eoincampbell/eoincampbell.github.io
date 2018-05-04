---
layout: post
title: Could not load type 'System.Runtime.CompilerServices. ExtensionAttribute' from
  assembly mscorlib when using ILMerge
date: 2013-03-06 13:05:09.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- C#
tags:
- ".net4"
- ".net4.0"
- ".net4.5"
- assembly
- framework
- ilmerge
- targetplatform
meta:
  _wpas_mess: Problems using ILMerge where both .NET 4.0 & .NET 4.5 have been installed
  _wpas_done_all: '1'
  _edit_last: '1'
  _wpas_skip_2206960: '1'
  _wpas_skip_2206956: '1'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1525095168;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:468;}i:1;a:1:{s:2:"id";i:789;}i:2;a:1:{s:2:"id";i:714;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p>[caption id="attachment_803" align="alignright" width="140"]<a href="http://trycatch.me/blog/wp-content/uploads/2013/03/worksonmymachine.png"><img class=" wp-image-803 " alt="Works On My Machine" src="{{ site.baseurl }}/assets/worksonmymachine.png" width="140" height="135" /></a> Works On My Machine[/caption]</p>
<p style="text-align: justify;">I ran into a pretty horrible problem with ILMerge this week when attempting to build and deploy a windows service I'd been working on. While the merged executable &amp; subsequently created MSI worked fine on my own machine, it gave the following rather nasty problem when run on a colleagues machine.</p>
<p style="text-align: justify;">[error]</p>
<p style="text-align: justify;">Could not load type 'System.Runtime.CompilerServices.ExtensionAttribute' from assembly mscorlib</p>
<p style="text-align: justify;">[/error]</p>
<p style="text-align: justify;">It turns out that between .NET 4.0 &amp; .NET 4.5; this attribute was moved from System.Core.dll to mscorlib.dll. While that sounds like a rather nasty breaking change in a framework version that is supposed to be 100% compatible, a [TypeForwardedTo] attribute is supposed to make this difference unobservable.</p>
<p style="text-align: justify;">Unfortunately things breakwhen ILMerge is used to merge several assemblies into one. When I merge my .NET 4.0 app, with some other assemblies on the machine with .NET 4.5 installed, it sets the targetplatform for ILMerge to .NET 4.0. This in turn looks into <span style="font-size: medium;"><strong><span style="font-family: 'courier new', courier;">C:\windows\Microsoft.NET\Framework\v4.0.30319</span></strong></span> to find the relevant DLLs. But since .NET 4.5 is an in place upgrade, these have all been updated with their .NET 4.5 counter parts.</p>
<p>[caption id="attachment_804" align="aligncenter" width="278"]<a href="http://xkcd.com/1172/"><img class="size-full wp-image-804" alt="Breaking Changes" src="{{ site.baseurl }}/assets/breaking-changes.png" width="278" height="386" /></a> "Every well intended change has at least one failure mode that nobody thought of"[/caption]</p>
<p style="text-align: justify;">You need to specific that ILMerge should use the older .NET 4.0 reference assemblies which are still available in <span style="font-size: medium;"><strong><span style="font-family: 'courier new', courier;">C:\Program Files\Reference Assemblies\Microsoft\Framework\.NETFramework\v4.0</span></strong></span>. (or program files x86) if your on a 64-bit box). There's more info on the stackoverflow question where I finally found a solution and in a linked blog post by Matt Wrock.</p>
<p style="text-align: justify;"><a href="http://stackoverflow.com/questions/13748055/could-not-load-type-system-runtime-compilerservices-extensionattribute-from-as" target="_blank">http://stackoverflow.com/questions/13748055/could-not-load-type-system-runtime-compilerservices-extensionattribute-from-a</a></p>
<p style="text-align: justify;">and</p>
<p style="text-align: justify;"><a href="http://www.mattwrock.com/post/2012/02/29/What-you-should-know-about-running-ILMerge-on-Net-45-Beta-assemblies-targeting-Net-40.aspx" target="_blank">http://www.mattwrock.com/post/2012/02/29/What-you-should-know-about-running-ILMerge-on-Net-45-Beta-assemblies-targeting-Net-40.aspx</a></p>
<p style="text-align: justify;">To override this behavior you need to specify this target platform directory as part of your ILMerge command. e.g.</p>
<pre class="brush:text;">"C:\Path\To\ILMerge.exe"
    /out:"$(TargetDir)OutputExecutable.exe"
    /target:exe
    /targetplatform:"v4,C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework\v4.0"
      "$(TargetDir)InputExecutable.exe"
      "$(TargetDir)A.dll"
      "$(TargetDir)B.dll"</pre>
<div style="text-align: justify;">I had previously been using the <a title="ILMerge.MSBuild.Tasks on Nuget" href="https://nuget.org/packages/ILMerge.MSBuild.Tasks/" target="_blank">ILMerge.MSBuild.Tasks tool from nuget</a> but unfortunately, this library doesn't currently support specifying the TargetPlatform. There's an<a title="IlMerge Tasks on Google Code" href="https://code.google.com/p/ilmerge-tasks/issues/detail?id=1" target="_blank"> unactioned open item on their issue tracker in google code</a>.</div>
<div></div>
<div><em>~EoinC</em></div>
