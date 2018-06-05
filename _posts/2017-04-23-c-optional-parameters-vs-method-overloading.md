---
layout: post
title: C# Optional Parameters vs. Method Overloading
date: 2017-04-23 11:47:36.000000000 +01:00
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
- c#
- method overload
- method overloading
- optional
- optional parameter
- overloading
- parameter
- polymorphism
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
<p>I was recently working on some data access/repository code and came across a nice generic repository implemetation for working with EntityFrameworks DataContexts. But when I ran our build through SonarQube, it got itself quite bent out of shape about the following recommendation.</p>
<blockquote><p>
Use the overloading mechanism instead of the optional parameters.</p></blockquote>
<p>Hmm... Why doesn't SonarQube like optional parameters. They've been a feature of C# since v4.0 and are extremely useful. Consider the following interface.<br />
In the first version, there's a single method definition that allows the consumer pick and choose what variety of parameters to pass.</p>
<p><script src="https://gist.github.com/eoincampbell/c95e4c9fb38cb83e42df2b60e5869a82.js"></script></p>
<p>In the second verion, you need 8 different method overloads to achieve the same.</p>
<p><script src="https://gist.github.com/eoincampbell/cb3f95fa71ef2756d3a4111e45796999.js"></script></p>
<p>So why is SonarQube recommending that latter over the former. Well there's a few reasons that may not be obvious. These methods are going to be public facing. They are the surface of your API and will be exposed to the world outside your DLL. In my case, my C# Service's were calling the repository in a different DLL. But not all consumers support Optional Parameters. Java and C++ do not have an equivalent concept available to them. Another reason is that these defaults do not "live" in your DLL per say, but get baked into the call site of the DLL that references them at compile time. So a change in the default values requires the caller application to be recompiled and redeployed rather than just an update of your API library.</p>
<p>Hence why the .NET Frameworks BCL libs all favour Method Overloading over Optional Parameters. But in my case this is internal project code, part of an application stack that gets deployed as a whole.</p>
<p>There's a benefit to brevity in my code and I'd prefer the implementation to be more concise since I having full control of both the API and Caller as well as the deployment process. So Optional Parameters it is.</p>
<p>If on the other hand you're writing public API libs for publishing to nuget or you operate in an environment where partial deployments of libraries and components occur, go with Method Overloading.</p>
<p><em>~Eoin Campbell</em></p>
