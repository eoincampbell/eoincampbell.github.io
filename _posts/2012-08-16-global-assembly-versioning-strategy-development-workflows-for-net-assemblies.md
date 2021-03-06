---
layout: post
title: Global Assembly Versioning Strategy & Development Workflows for .NET Assemblies
date: 2012-08-16 10:15:13.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- C#
- Coding
tags:
- ".net"
- assemblies
- assembly
- assembly version
- attribute
- batch command
- Development
- gac
- gacutil
- global assembly cache
- Programming
- version
- version number
- workflow
meta:
  _edit_last: '1'
  _thumbnail_id: '474'
  _wpas_skip_2206960: '1'
  _wpas_skip_2206956: '1'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1525017936;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:614;}i:1;a:1:{s:2:"id";i:802;}i:2;a:1:{s:2:"id";i:789;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p style="text-align: justify;"><img class="alignright size-thumbnail wp-image-474" title="v1.0" alt="Version 1.0" src="{{ site.baseurl }}/assets/v1.0-150x150.jpg" width="150" height="150" />Over the past few weeks I've been working on a new project in our company which involved building a number of inter-dependent assemblies, "strongly naming" them and installing them into the Global Assembly Cache. Over the course of the project, I was forced to look at a number of issues related to assembly versions, solution organisation and the deployment of assesmblies in a developer environment.</p>
<p style="text-align: justify;">So given that it's been a while since I wrote anything vaguely technical, I thought I'd document some of these issues down.</p>
<ul style="text-align: justify;">
<li>What version numbering strategy should we use?</li>
<li>How will we organise our Solution to make this easily manageable?</li>
<li>How will we manage these libraries during the deployment phase?</li>
<li>How will we circulate stable versions to developers during on-going development of other projects?</li>
<li>How will we release these libraries to customers?</li>
</ul>
<div style="text-align: justify;"><strong><span style="font-size: small;">[notice]TL;DR - Take a look at the <a title="VersioningDemo Solution on GitHub" href="https://github.com/eoincampbell/versioning-demo" target="_blank">VersioningDemo Solution on GitHub</a>[/notice]</span></strong></div>
<p style="text-align: justify;"><!--more--></p>
<h3 style="text-align: justify;"><span style="text-decoration: underline;"><strong>Assembly Version Attributes in .NET Assemblies</strong></span></h3>
<p style="text-align: justify;">Before we start it's probably worth spending a little time on Assembly Versions in .NET. Microsoft .NET supports 3 different types of "Version Number" information on an Assembly. Descriptions below are taken from MSDN.</p>
<ol style="text-align: justify;">
<li><a title="AssemblyVersionAttribute" href="http://msdn.microsoft.com/en-us/library/system.reflection.assemblyversionattribute.aspx" target="_blank">Assembly Version</a>; The assembly version number is part of an assembly's identity and plays a key part in binding to the assembly and in version policy. The default version policy for the runtime is that applications run only with the versions they were built and tested with. (Unless overridden in configuration).</li>
<li><a title="AssemblyFileVersionAttribute" href="http://msdn.microsoft.com/en-us/library/system.reflection.assemblyfileversionattribute.aspx" target="_blank">Assembly File Version</a>; A specific version number for the Win32 file version resource. The Win32 file version is not required to be the same as the assembly's version number, buf if not supplied, the AssemblyVersionAttribute is used. It can be seen on the Version tab of the Windows file properties dialog.</li>
<li><a title="AssemblyInformationalVersionAttribute" href="http://msdn.microsoft.com/en-us/library/system.reflection.assemblyinformationalversionattribute.aspx" target="_blank">Assembly Informational Version</a>; An additional version information for an assembly manifest. If the AssemblyInformationalVersionAttribute is not applied to an assembly, the version number specified by the Assembly Version Attribute attribute is used instead.</li>
</ol>
<p style="text-align: justify;">These 3 attributes can be set on an assembly in the $Project\Properties\AssemblyInfo.cs file</p>

```csharp
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyVersion("1.0.0.*")] //Auto Revision Number
[assembly: AssemblyVersion("1.0.*")]   //Auto Build &amp; Revision Numbers

[assembly: AssemblyFileVersion("1.0.0.0")]

[assembly: AssemblyInformationalVersion("1.0.0.0")];
```

<p style="text-align: justify;">For the AssemblyVersionAttribute we have the option of replacing the standard <em>{major}.{minor}.{build}.{revision}</em> format, with either <em>{major}.{minor}.{build}.*</em> or <em>{major}.{minor}.* </em>which will cause the compiler to automatically generate the build and revision components. If we do choose to use the automatic build and revision numbers, the <em>{build} </em>number will default to the number of days since January 1st, 2012. The <em>{revision} </em>number will default to the number of seconds since midnight, divided by 2.</p>
<p><a href="http://msdn.microsoft.com/en-us/library/system.reflection.assemblyversionattribute.aspx">http://msdn.microsoft.com/en-us/library/system.reflection.assemblyversionattribute.aspx</a></p>
<p><strong>Creating a Strong Name Key for your Assemblies </strong></p>
<p style="text-align: justify;">In order to place a .NET Assembly into the GAC, the assembly must be "Strongly Named". A strong name is an identifer for the assembly comprising of  simple text name, version number, and culture information (if provided) and a public key and a digital signature. In order to strongly name your assembly we need to generate a Strong-Name Key. This can be done from inside Visual Studio, but this will make the key a part of that specific project. My personal preference is to generate the key externally using the sn.exe tool. This makes it a little easier to store the key else where and share it among other projects as we'll see below.</p>

```
sn -k keyPair.snk
```

<h3><span style="text-decoration: underline;"><strong>Solution Organisation</strong></span></h3>
<p style="text-align: justify;">So lets start creating the solution. The <a title="VersioningDemo Solution on GitHub" href="https://github.com/eoincampbell/versioning-demo">sample solution is available here from GitHub</a> which shows the final product. First we create 2 Class Library projects and make one a reference of the other. We also create a "Solution Items" folder for storing common items such as our Strong Name Key and common assembly info.</p>
<p><img class=" wp-image-480 " title="Solution Organization" alt="Solution Organization" src="{{ site.baseurl }}/assets/sol1.png" width="238" height="160" /> Solution Organization</p>
<p style="text-align: justify;">We also update the output path for all assembly configurations to just <span style="text-decoration: underline;"><strong>bin\</strong></span>. This makes it easier to manage Debug/Release versions in post-build batch scripts as only one version can exist simultaneously.</p>
<p><img class=" wp-image-490 " title="Common bin\ Output" alt="Common bin\ Output" src="{{ site.baseurl }}/assets/sol6.png" width="449" height="146" /> Common bin\ Output</p>
<h4><strong>Common Strong Name Key File</strong></h4>
<p style="text-align: justify;">Next we create our Strong Named Key. In order to use the same key on each of our assemblies we externally generate an snk file using the following command and place it in an windows explorer folder under the solution.</p>

```
sn -k VersioningDemo.snk
``` 

<p style="text-align: justify;">This file is placed in the root solution folder in windows explorer and then dragged into the Solution Items Folder within the Solution. Finally, we add this snk file to each individual project as a "Linked" File.</p>
<p><img class=" wp-image-482 " title="Adding a Strong Named Key as a Linked File" alt="Adding a Strong Named Key as a Linked File" src="{{ site.baseurl }}/assets/sol2.png" width="527" height="506" /> Adding a Strong Named Key as a Linked File</p>
<p style="text-align: justify;">Once the strong named key is added as a linked file, we configure the assembly to be signed-on-build in the project properties screen.</p>
<p><img class=" wp-image-483 " title="Signing The Assembly" alt="Signing The Assembly" src="{{ site.baseurl }}/assets/sol3.png" width="463" height="139" /> Signing The Assembly</p>
<h4><strong>Common "GlobalAssemblyInfo" File</strong></h4>
<p style="text-align: justify;">Each project comes with a default Assembly Info File under its property folder. In large solutions, however, maintaining all these different assembly info files can be unwieldy, especially if there are common attributes that need to be shared/synchronized between the different projects. We can achieve this by adding a shared "GlobalAssemblyInfo" file to the solution. Adding the file to each project is done in the same way as above for snk. We create a GlobalAssemblyInfo.cs in the solution root folder in Windows Explorer, add it to the Solution Items folder inside the SLN, and then add it to each individual project as a linked file. Once added, it can be dragged under the Properties folder to keep things organised.</p>
<p><img class=" wp-image-484 " title="Organised Solution " alt="Organised Solution" src="{{ site.baseurl }}/assets/sol4.png" width="366" height="338" /> Organised Solution</p>
<p style="text-align: justify;">The purpose of having these two files is that we can now split out the different assembly attributes that are specific to our individual projects, from the ones that we want to keep common across the solution. In the sample solution the attributes have been divided over the two files in the following way.</p>
<p><strong>Global Assembly Info</strong></p>

```csharp
using System.Reflection;
#if DEBUG
[assembly: AssemblyConfiguration("Debug")]
#else
[assembly: AssemblyConfiguration("Release")]
#endif
[assembly: AssemblyCompany("My Company")]
[assembly: AssemblyCopyright("Copyright © My Company 2012")]
[assembly: AssemblyTrademark("My Trademark ™")]
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")]
[assembly: AssemblyInformationalVersion("1.0.0.0")]
```
<p><strong>Project Assembly Info</strong></p>

```csharp
using System.Reflection;
using System.Runtime.InteropServices;
[assembly: AssemblyTitle("VersioningDemo.DependentLib")]
[assembly: AssemblyDescription(
           "The DependentLib Library of the VersioningDemo Solution")]
[assembly: AssemblyProduct("VersioningDemo.DependentLib")]
[assembly: AssemblyCulture("")]
[assembly: ComVisible(false)]
[assembly: Guid("fc4e547c-1797-4381-a957-d024abbf2f26")]
```
<h3><span style="text-decoration: underline;"><strong>Version Number Strategy</strong></span></h3>
<p><a id="vs">&nbsp;</a></p>
<p style="text-align: justify;">Phew! Getting there... so now that the solution is organised, we need to decide what version numbering scheme/strategy to use to version our Assemblies. What other considerations need to be taken? This is a very open ended subject and will most likely be dependent on your own company or situation. You should bear the following in mind.</p>
<ol style="text-align: justify;">
<li style="text-align: justify;">Try and stick to the <em>{major}.{minor}.{something}.{something}</em> versioning mechanism. It's not necessary to use build numbers and software revisions but following a general major/minor verison number pattern will help your team, sales people and customers alike recognise where on the product version ladder they are.</li>
<li style="text-align: justify;">Remember that changing the AssemblyVersion number will mean having to recompile/re-release every other application that references that assembly.</li>
<li style="text-align: justify;">Remember that changing the Assembly File Version number will not. This can freely change between releases.</li>
</ol>
<p style="text-align: justify;">In my company, we use a numbering scheme of {Year}.{Quarter}.{Month_Of_Quarter}.{Revision} as we only do quarterly stable builds of our core assemblies. In my own personal projects, I use a standard incrementing {major}.{minor} scheme with current {day_of_year_number * year} as the {build} portion of the Assembly and AssemblyFile version numbers. And the .Net standard of current second in the day divided by 2 for the revision. This is easily gotten using this quick powershell script.</p>
```
PS C:\> [int32](((get-date).Year-2000)*366)+(Get-Date).DayOfYear
4781
PS C:\> [int32]((get-date)-(Get-Date).Date).TotalSeconds
62595
PS C:\>
# 4781 = ((2013-2000) * 366) + 23 = Wednesday January 23rd, 2013
```
<p style="text-align: justify;">The revision component of the AssemblyVersion is left out, but in the Assembly File Version it's set to the current Source Control ChangeSet Revision at the time of build. Finally the Informational Version is set to a more verbose description of the current software production version.</p>

```csharp
[assembly: AssemblyVersion("1.0.2736")]
[assembly: AssemblyFileVersion("1.0.2736.67")]
[assembly: AssemblyInformationalVersion("VersioningDemo v1.0.2736.67 RC1, Hotfix B")]
```
<p style="text-align: justify;">[notice]So why go to all this trouble?</p>
<p style="text-align: justify;">Well this is a very powerful combination of information. When we lock down a release and roll it out, the AssemblyVersion is committed as part of the assembly strong name and once installed in the GAC, any applications compiled against this version are tied to that AssemblyVersion.</p>
<p style="text-align: justify;">But imagine we find a bug. It doesn't change any interface contracts or methods but it needs to be fixed with a hotfix release. No problem, we can rebuild and re-release the assembly with the same AssemblyVersion, and the applications will continue to happily reference it. We can also bump the AssemblyFileVersion so we can see at a glance which specific version is in place at a particular deployment site. We can view the File Version and Informational Version through the Windows Explorer Properties window.[/notice]</p>
<p><img class="size-full wp-image-488" title="Assembly Properties" alt="Assembly Properties" src="{{ site.baseurl }}/assets/sol5.png" width="320" height="260" /> Assembly Properties</p>
<h3><span style="text-decoration: underline;"><strong>Development Workflow</strong></span></h3>
<p style="text-align: justify;">Now that we have a stable build of our libs we have a few things to consider relating to our development process and developer workflow. Let's take each of the following, one at a time.</p>
<ul>
<li>Since we intend to add these libs to the GAC, how can we build &amp; debug them in that same context?</li>
<li>How will we distribute Stable Versions to other Developers?</li>
<li>How will we keep other developers in Sync?</li>
</ul>
<h4>Auto-GAC'ing Assemblies</h4>
<p style="text-align: justify;">Visual Studio comes with an SDK Tool called GACUtil for adding Assemblies to the Global Assembly Cache. It's important to note that GACUtil is not a redistributable application and <strong>should NOT </strong>be used for adding assemblies to the GAC in a production environment, but in "Developer-land" we are free to leverage it. After each build we can automatically place each assembly in the global assembly cache using a Post-Build Command on the Project. (Project -&gt; Properties -&gt; Build Events -&gt; Post-Build Event)</p>

```
"C:\Program Files (x86)\Microsoft SDKs\Windows\v7.0A\Bin\NETFX 4.0 Tools\gacutil.exe" /i "$(TargetPath)"
```

<p style="text-align: justify;">Even if we install the assemblies from the "Stable Build" Repository (which we'll talk about next), each subsequent build of the VersioningDemo Solution on our own machine will supersede those references and replace them in our GAC (so long as we haven't updated our AssemblyVersion Number).</p>
<h4 style="text-align: justify;">The "Stable Builds" Repository</h4>
<p style="text-align: justify;">Once a valid "cut" of the assemblies has been taken for distribution (i.e. a stable branch or tag or whatever the in-house process might be), we'll complete a release build of our Application and then move our assemblies to a "Stable Builds" folder within our Source Control System. The solution contains a folder called "Stable" under the root of the solution folder. Beneath this, a directory is created for each stable build and named with that build's assembly version number.</p>
<p><img class=" wp-image-491 " title="Stable Folder Structure" alt="Stable Folder Structure" src="{{ site.baseurl }}/assets/sol7.png" width="419" height="190" /> Stable Folder Structure</p>
<p style="text-align: justify;">To help manage this we can use the following <strong>CopyToStable.bat</strong> batch file which pushes the latest compiled assemblies from the \bin\ directory to a specified stable version folder.</p>

```
@ECHO OFF
SET /P GACVersion=Enter the Assembly Version Number for this cut: 
IF '%GACVersion%'=='' (
	ECHO You must enter a version number.
	PAUSE
	GOTO:EOF
)

SET DestDir="%CD%\Stable\%GACVersion%"
IF NOT EXIST %DestDir% MKDIR %DestDir%
DEL /S /F /Q %DestDir%*.*

COPY %CD%\VersioningDemo.Core\bin\*.dll %DestDir%
COPY %CD%\VersioningDemo.Core\bin\*.pdb %DestDir%
COPY %CD%\VersioningDemo.Core\bin\*.xml %DestDir%
COPY %CD%\VersioningDemo.DependentLib\bin\*.dll %DestDir%
COPY %CD%\VersioningDemo.DependentLib\bin\*.pdb %DestDir%
COPY %CD%\VersioningDemo.DependentLib\bin\*.xml %DestDir%

ECHO Done!
PAUSE
```

<h4 style="text-align: justify;">Keeping Other Developers "In-Sync" via SVN</h4>
<p style="text-align: justify;">Finally we need to circulate these libraries to the other developers in the team. This is accomplished by checking the Stable folder into SVN. We have a developer team svn script which we use to pull the latest head revision from trunk each morning (or the relevant branches that a developer might be working on). At the bottom of this script we've included some additional batch commands to automatically iterate all the subfolders beneath each Stable Build Directory and automatically re-install the assemblies to the GAC. Considerations:</p>
<ul style="text-align: justify;">
<li>Since "Stable" is committed in the same tree structure as the solution itself, branching the solution also branches the Stable Libs.</li>
<li>Running the "GetTrunk" svn script will auto check-out and GACUtil the latest versions of the Stable Libs from Trunk</li>
<li>Running one of the "GetBranch" svn scripts will auto check-out and GACUtil the latest versions of that branch superseding what was previously installed to the GAC.</li>
<li>Opening and re-compiling the GAC Assembly Solution locally will automatically add those newly compiled assemblies to the GAC, superseding what was previously installed.</li>
</ul>

```
@echo off
echo Updating GAC. Please wait...
echo GAC Update on %date% at %time% &gt; GACLog.txt
echo Stopping IIS for GAC registration...
iisreset /STOP &gt; nul
dir /B /S Stable\*.dll &gt; AssemblyList.txt
echo ---------------- AssemblyList.txt ------------- &gt;&gt; GACLog.txt
"C:\Program Files (x86)\Microsoft SDKs\Windows\v7.0A\Bin\NETFX 4.0 Tools\gacutil.exe" /il AssemblyList.txt /f &gt;&gt; GACLog.txt
echo ---------------- AssemblyList.txt --------------- &gt;&gt; GACLog.txt
echo Deleting contents of download cache...
"C:\Program Files (x86)\Microsoft SDKs\Windows\v7.0A\Bin\NETFX 4.0 Tools\gacutil.exe" /cdl &gt;&gt; GACLog.txt /nologo
del AssemblyList.txt 2&gt;&gt; GACLog.txt
echo Restarting IIS...
iisreset /START &gt; nul
echo Done.
echo Please see 'GACLog.txt' for details of GAC Install.
pause
```
<p style="text-align: justify;">So there you have it. My current solution for managing and versioning Strong-named, GAC'd assemblies in a Development Workflow. If anyone has any feedback, recommendations or things they do differently, I'd love to hear about them.</p>
<p style="text-align: justify;"><strong><em>~Eoin Campbell</em></strong></p>
