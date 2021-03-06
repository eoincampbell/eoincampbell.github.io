---
layout: post
title: Combinatorics in .NET - Part II - Creating a Nuget Package
date: 2012-09-05 15:34:35.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- C#
- Coding
- Visual Studio
tags:
- ".net"
- ".net4.5"
- assembly
- c#
- combination
- combinatorics
- github
- library
- nuget
- nuspec
- package
- permuation
- solution
- variation
meta:
  _edit_last: '1'
  _thumbnail_id: '642'
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><img class=" wp-image-643  " title="nuget.org" src="{{ site.baseurl }}/assets/nuget1.png" alt="nuget.org" width="188" height="60" /> nuget.org</p>
<p>[important]<br />
This is part 2 of a 2 part post on Combinatorics in .Net</p>
<p>The solution is publicly available on github; <a title="Combinatorics Solution on GitHub" href="https://github.com/eoincampbell/combinatorics" target="_blank">https://github.com/eoincampbell/combinatorics</a></p>
<p>The library can be added to any .NET Soution via Nuget; <a title="Combinatorics Package on Nuget" href="https://nuget.org/packages/Combinatorics" target="_blank">https://nuget.org/packages/Combinatorics</a><br />
[/important]</p>
<p style="text-align: justify;">In <a title="Combinatorics in .NET – Part I – Permutations, Combinations &amp; Variations" href="http://trycatch.me/combinatorics-in-net-part-i-permutations-combinations-variations/" target="_blank">the last post</a> we looked at the Combinatorics Library, a .NET Assembly which provides Combinatoric generation capabilities to your .NET Applications. Now lets look at bundling up that solution &amp; deploying the package to Nuget. Nuget is an online .NET Package Repository &amp; associated Visual Studio extension that makes it easy to manage external assemblies in your projects. Developers who build 3rd party libraries or tools can create a NuGet package and store the package in a NuGet repository. Other developers can then browse the repository (online or with Visual Studio) and add references to those 3rd party tools &amp; libraries.</p>
<p style="text-align: justify;">You can read more about Nuget <a title="Nuget Overview" href="http://docs.nuget.org/docs/start-here/overview" target="_blank">here</a>.</p>
<p style="text-align: justify;"><!--more--></p>
<h1 style="text-align: justify;">Solution Organisation</h1>
<p style="text-align: justify;">Nuget Packages support bundling different versions of an assembly based on different target frameworks and different dependencies. Since the combinatorics libraries rely only on .NET 2.0 Generics one project within the solution is set to target .NET Framework 2.0 as a minimum. A second project currently contains the same Combinatorics functionality but is set up as a new "<a title="Portable Library Project - MSDN" href="http://msdn.microsoft.com/en-us/library/gg597391.aspx" target="_blank">Portable Class Library</a>" Project format. This project type allows you to specify the minimum frame work requirements for the .NET Framework, Silverlight, Windows RT &amp; the Xbox Libraries. The Combinatorics.net40 project targets .NET4.0, Silverlight 4 &amp; WinRT and will allow for improvement of the library using BCL functionality available in those frameworks.</p>
<h1 style="text-align: justify;">Creating a Nuspec</h1>
<p style="text-align: justify;">In order to deploy an assembly (or set of assemblies) as a NugetPackage, you need to first fill out a Nuspec file for that deployment. Think of a Nuspec as a sort of manifest or metadata file that contains information about the package. First of all we generated a template specification file using the following command</p>
<blockquote><p><code>nuget spec Combinatorics.dll</code></p></blockquote>
<p>&nbsp;</p>

```xml
<?xml version="1.0"?>
<package >
    <metadata>
        <id>Combinatorics</id>
        <version>1.0.3.2</version>
        <title>Combinatorics Library for .NET</title>
        <authors>Adrian Akison, Eoin Campbell</authors>
        <owners>Eoin Campbell</owners>
        <licenseUrl>https://github.com/eoincampbell/combinatorics/blob/master/LICENCE</licenseUrl>
        <projectUrl>https://github.com/eoincampbell/combinatorics</projectUrl>
        <iconUrl>https://raw.github.com/eoincampbell/combinatorics/master/combinatorics.png</iconUrl>
        <requireLicenseAcceptance>false</requireLicenseAcceptance>
        <summary>A Combinatorics Library for Combinations, Permutations &amp;amp; Variations in .NET</summary>
        <description>A Combinatorics Library providing Generic IEnumerable classes ...</description>
        <releaseNotes>Notes</releaseNotes>
        <copyright>Copyright © 2012</copyright>
        <tags>Combinatorics Combination Permutation Variation .NET Library</tags>
    </metadata>
</package>
```

<p style="text-align: justify;">Once the specification file was created, it was moved to a "preparation folder". Nuget supports spec'ing and packaging an assembly direcltly from the Project directory &amp; csproj files, but if you want to do your own multi-framework targetting you need to manage things a little differently. The "Nuget" folder under the solution root contains the Spec &amp; also contains the required <em><strong>Lib</strong></em> folder structure for the different versions of the assemblies.</p>
<p style="text-align: justify;">You can read more about the internal organipackage</p>
<h1 style="text-align: justify;">Building &amp; Deploying the Package</h1>
<p style="text-align: justify;">Building the Solutions &amp; Deploying the package to nuget is handled by 2 batch files in the solution root.</p>
<p style="text-align: justify;">First <strong>Build.bat </strong>runs, builds both solutions, cleans out the old Nuget Package Lib structure &amp; copies the new assemblies to the newly created directory structure.</p>

```
@echo off
devenv Combinatorics.sln /Clean
devenv Combinatorics.sln /Build Debug

if exist Nuget\lib rmdir /S /Q Nuget\lib

mkdir Nuget\lib
mkdir Nuget\lib\net20
mkdir Nuget\lib\net40
mkdir Nuget\lib\sl40

copy "Combinatorics.Net20\bin\*.dll" "Nuget\lib\net20\"
copy "Combinatorics.Net20\bin\*.pdb" "Nuget\lib\net20\"
copy "Combinatorics.Net20\bin\*.xml" "Nuget\lib\net20\"

copy "Combinatorics.Net40\bin\*.dll" "Nuget\lib\net40\"
copy "Combinatorics.Net40\bin\*.pdb" "Nuget\lib\net40\"
copy "Combinatorics.Net40\bin\*.xml" "Nuget\lib\net40\"

copy "Combinatorics.Net40\bin\*.dll" "Nuget\lib\sl40\"
copy "Combinatorics.Net40\bin\*.pdb" "Nuget\lib\sl40\"
copy "Combinatorics.Net40\bin\*.xml" "Nuget\lib\sl40\"
```

<p style="text-align: justify;">Next the nupkg package is created &amp; deployed to Nuget using the PushToNuget.bat BatchFile.</p>

```
@echo off
nuget pack Nuget\Combinatorics.nuspec
nuget push %1
```

<p style="text-align: justify;">Magic, we've just deployed our first nuget package. You can download the combinatorics package from Nuget <a title="Combinatorics on Nuget" href="https://nuget.org/packages/Combinatorics" target="_blank">here</a> and read more about the using the solution <a title="Combinatorics in .NET – Part I – Permutations, Combinations &amp; Variations" href="http://trycatch.me/combinatorics-in-net-part-i-permutations-combinations-variations/" target="_blank">here</a> &amp; <a title="Combinatorics on Eoin Campbell's GitHub" href="https://github.com/eoincampbell/combinatorics" target="_blank">here</a>.</p>
<p><em>~Eoin Campbell</em></p>
