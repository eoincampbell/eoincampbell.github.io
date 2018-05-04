---
layout: post
title: Appveyor Continuous Integration and Continuous Delivery
date: 2017-05-01 10:52:00.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- Appveyor
- Coding
- Environments &amp; Tools
tags:
- appveyor
- cd
- ci
- continuous
- continuous delivery
- continuous deployment
- continuous integration
- delivery
- deployment
- integration
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
<p>I've been maintaining/curating a <a href="https://www.nuget.org/packages/Combinatorics/" target="_blank">nuget package for combinatorics</a> over on <a href="https://github.com/eoincampbell/combinatorics" target="_blank">Github</a> for a while, which I decided to upgrade to .NET Standard over the weekend. I'd noticed that lots of other FOSS projects were up and running on a platform called <a href="http://appveyor.com" target="_blank">AppVeyor</a> and had nice little <i>build passing</i> checks on them like this <img src="{{ site.baseurl }}/assets/tr38vj9ebhokwsyi?svg=true" alt="build-passing" /> so I decided to suss it out.</p>
<p>Appveyor is a cloud hosted continuous integration & continuous delivery platform. We use various CI and CD tools at work like Jenkins & Octopus which require a little bit of setting up so I wasn't really sure what to expect. I certainly wasn't expecting it to be this easy. </p>
<p><img src="{{ site.baseurl }}/assets/AppveyorBuild-1024x337.png" alt="Appveyor Build" width="740" height="244" class="aligncenter size-large wp-image-1403" /></p>
<p>Signing up was a matter of creating a free account using my Github credentials (AppVeyor is free for open-source projects/public repos). Then it will prompt you to select the projects you'd like to pull in for CI. I chose the Combinatorics project. Next it figured out from the presence of the Solution (.sln) file, and two unit test projects (MSTest) within, that this was a .NET project and it auto-generated an MSBuild CI job, including automated test runs. Boom! CI Pipeline Done. I'm suitably impressed.</p>
<p>Beyond the vanilla setup it seems quite extensive in its capabilities:</p>
<ul>
<li>You can control which branches/tags and pull requests automatically trigger builds as well as triggering them yourself via web-hook or cron schedule</li>
<li>It allows you to customise your entire build pipeline by inserting scripts pre build, pre package, post build, on success, on fail through out for custom jobs execution.</li>
<li>You can be notified in a variety of different ways including email, web-hook or slack</li>
<li>It has built in transformation processes for AssemblyInfo files so you can tie your build and version info together</li>
<li>You can add custom artifacts to include in deployments</li>
<li>It supports a whole variety of target deployments for your project artifacts including Azure, FTP, Nuget and more</li>
</ul>
<p>There's probably more I haven't discovered yet. Next job will be to automate the push to Nuget based on commits back to master after Pull Requests are confirmed.</p>
<p><em>~Eoin C</em></p>
