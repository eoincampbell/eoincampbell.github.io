---
layout: post
title: Publish a PowerShell module to MyGet in 5 Minutes
date: 2017-12-02 22:52:27.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Coding
- Powershell
tags:
- Module
- MyGet
- Powershell
- Publish
meta:
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1525303671;s:7:"payload";a:0:{}}}
  _edit_last: '1'
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p>I've been trying to sell my colleagues on the benefits of PowerShell. Need to interact with Git, do it from PowerShell. Got some repetitive task to perform? PowerShell. I'd like a cup of coffee... seriously <a href="https://p0wershell.com/?p=5491">write a script to contact your IOT enabled coffee machine</a> and have it ready for you when you get there.</p>
<p><img src="{{ site.baseurl }}/assets/powershell-all-the-things.jpg" alt="PowerShell All The Things" width="772" height="355" class="aligncenter size-full wp-image-1433" /></p>
<p>A buddy was doing some QA testing of a GeoIP lookup service we've built. We store IPs as unsigned ints (for speedy Azure table storage row-key lookup) and he needed a quick way to go from integer to 4-octet (1.2.3.4) and back. So we knocked up some <a href="https://github.com/eoincampbell/powershell-scripts/blob/master/EoinCModule/EoinCModule.psm1">PowerShell functions</a>. And then a train of thought took over.</p>
<p>Everyone should hopefully be in the mindset of automating and scripting common tasks. But the benefit is even more powerful if you can share and distribute these things among your friends and colleagues. Maybe you do already. Maybe you paste scripts into Slack. Maybe you have a Github repo that people can grab them from and add import statements to their PSProfile. Here, we're going to look at bundling our module into a package and deploying it to MyGet so it can be shared and installed by the masses.</p>
<p>Seems like a good idea... right?</p>
<h2>Create a PowerShell module</h2>
<p>First, I created a new module in my <a href="https://github.com/eoincampbell/powershell-scripts/tree/master/EoinCModule">github repo of powershell scripts</a>, creatively named <strong>EoinCModule</strong>.</p>
<p>The Module directory contains 2 files, the module itself, and a manifest file.</p>
<ul>
<li>EoinCModule
<ul>
<li>EoinCModule.psm1</li>
<li>EoinCModule.psd1</li>
</ul>
</li>
</ul>
<p>A PowerShell module, is a PSM1 file which contains your functions and commandlets. It's a pretty straight forward PowerShell script file with parameters, comments etc...</p>
<p>The manifest file contains meta-data about the module to describe it during the publish process. To create a template manifest file, you can run the following command in your module directory.</p>
```powershell
New-ModuleManifest -Path EoinCModule.psd1
```
<p>The manifest file contains the meta data about your module. At a minimum you'll need to update it to include</p>
<ul>
<li>Author</li>
<li>Description</li>
<li>ModuleVersion</li>
<li>RootModule (your PSM1 file)</li>
<li>FunctionsToExport</li>
</ul>
<h2>Signup to MyGet</h2>
<p>Next you'll need somewhere to publish your package to. <a href="https://www.myget.org/company">MyGet</a> is a public package management site that's free to sign up for.</p>
<p>I signed up with my gmail account, created a <a href="https://www.myget.org/feed/Packages/eoinc">public feed</a> and that's it.</p>
<h2>Publish your Packages</h2>
<p>PowerShell 5.0 has integrated publish support to allow you to publish direct to MyGet.</p>
<p>In MyGet, navigate to your <a href="https://www.myget.org/profile/Me#!/AccessTokens">profile page</a> and grab your Publisher API key.</p>
<p>Once you have your API Key, run the following commands from your PowerShell prompt to publish your module. (Don't forget to replace your feed name &amp; api key)</p>
```powershell
Import-Module PowerShellGet
$PSGalleryPublishUri = 'https://www.myget.org/F/YOUR-FEED-NAME/api/v2/package'
$PSGallerySourceUri = 'https://www.myget.org/F/YOUR-FEED-NAME/api/v2'
$APIKey = 'YOUR-API-KEY'

Register-PSRepository -Name MyGetFeed -SourceLocation $PSGallerySourceUri -PublishLocation $PSGalleryPublishUri
Publish-Module -Path .\YOUR-MODULE-DIRECTORY -NuGetApiKey $APIKey -Repository MyGetFeed -Verbose
```
<p>That's it. Your package is published.</p>
<h2>Importing your package</h2>
<p>Once your package is published you (or anyone else) can import it using the following command.</p>
```
Install-Module -Name "MODULE-NAME" -RequiredVersion "VERSION" -Repository "REPO-NAME"
```
<h2>Other Bits &amp; Pieces</h2>
<p>The process outlined above is pretty trivial, and if you have some module scripts already, getting them published shouldn't take any longer than 5-10 minutes. I did run into a couple of little issues though. When you generate your manifest, the default template will contain a version number with only a Major.Minor component. In order to publish, the version number must contain at BUILD component as well.</p>
<p>In order to publish you'll notice a command above to register a <code>PSRepository</code>. This will persist across PowerShell sessions once created. In the above example, I named my repository "MyGetFeed".</p>
<p>To install my module for example, you would run</p>
```powershell
Install-Module -Name "EoinCModule" -RequiredVersion "1.0.0" -Repository "MyGetFeed"
```
<p>To install my module on a different machine, you would first need to register the repository on that machine. e.g.</p>
```
Import-Module PowerShellGet
Register-PSRepository -Name "eoinc" -SourceLocation "https://www.myget.org/F/eoinc/api/v2"
Install-Module -Name "EoinCModule" -RequiredVersion "1.0.0" -Repository "eoinc"
```
<p><em>~Eoin Campbell</em></p>
