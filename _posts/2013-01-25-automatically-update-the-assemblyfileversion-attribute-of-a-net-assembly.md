---
layout: post
title: Automatically update the AssemblyFileVersion attribute of a .NET Assembly
date: 2013-01-25 10:11:26.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- C#
- Coding
- Powershell
- Visual Studio
tags:
- ".net"
- assembly
- assembly version
- assemblyfileversion
- automated
- automatic
- Powershell
- pre build
- script
- version
meta:
  _edit_last: '1'
  _wpas_done_all: '1'
  _wpas_skip_2206960: '1'
  _wpas_skip_2206956: '1'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1525376158;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:468;}i:1;a:1:{s:2:"id";i:614;}i:2;a:1:{s:2:"id";i:802;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><img class=" wp-image-474 " title="Automatic AssemblyFileVersion Updates" alt="Automatic AssemblyFileVersion Updates" src="{{ site.baseurl }}/assets/v1.0.jpg" width="162" height="162" />Automatic AssemblyFileVersion Updates</p>
<p style="text-align: justify;">There is support in .NET for automatically incrementing the AssemblyVersion of a project by using the <strong>".*" </strong>notation. e.g.<br />
<code>[assembly: AssemblyVersion("0.1.*")]</code></p>
<p style="text-align: justify;">Unfortunately the same functionality isn't available for the AssemblyFileVersion. Often times, I don't want to bump the AssemblyVersion of an assembly as it will effect the strong name signature of the assembly, and perhaps the changes (a bug fix) isn't significant enough to warrant it. However I do want to automatically increment the file version, so that in a deployed environment, I can right click the file and establish when the file was built &amp; released.</p>
<p style="text-align: justify;">[important]Enter the <a title="Update-AssemblyFileVersion.ps1 on Github" href="https://github.com/eoincampbell/powershell-scripts/blob/master/Update-AssemblyFileVersion.ps1" target="_blank">Update-AssemblyFileVersion.ps1</a> file.[/important]</p>
<p style="text-align: justify;">This powershell script, (<a title="Automatically Set the AssemblyFileVersion for Visual Studio Projects" href="http://blog.davidjwise.com/2012/07/12/automatically-set-the-assemblyfileversion-for-visual-studio-projects/" target="_blank">heavily borrowed from David J Wise's article</a>), runs as a pre-build command on a .NET Project. Simply point the command at an assembly info file, (or GlobalAssemblyInfo.cs if you're following <a title="Global Assembly Versioning Strategy &amp; Development Workflows for .NET Assemblies" href="http://trycatch.me/global-assembly-versioning-strategy-development-workflows-for-net-assemblies/" target="_blank">my suggested versioning tactics</a>)  and ta-da, automatically updating AssemblyFileVersions.</p>
<p style="text-align: justify;">The Build component of the version number will be set using the following formula based on a daycount since the year 2000.</p>

```
# Build = (201X-2000)*366 + (1==>366)
#
    $build = [int32](((get-date).Year-2000)*366)+(Get-Date).DayOfYear
```

<p style="text-align: justify;">The Revision component of the version number will be using the following formula based on seconds in the current day.</p>

```
# Revision = (1==>86400)/2 # .net standard
#
    $revision = [int32](((get-date)-(Get-Date).Date).TotalSeconds / 2)
``` 

<p style="text-align: justify;">The Major &amp; Minor components are not set to update although they could be. Simply add the following command to your Pre-Build event and you're all set.</p>

```
%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe 
    -File "C:\Path\To\Update-AssemblyFileVersion.ps1"  
    -assemblyInfoFilePath "$(SolutionDir)\Project\Properties\AssemblyInfo.cs"
```

<p><em>~Eoin C</em></p>
