---
layout: post
title: ASP.NET MVC 3 Beta & Razor View Engine
date: 2010-10-10 15:09:15.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- ASP.NET MVC
- Coding
tags:
- ASP.NET
- Beta
- Javascript
- MVC
- MVC3
- Razor
- Unobtrusive
meta:
  _edit_last: '1'
  _wp_old_slug: ''
  _thumbnail_id: '202'
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><a href="http://trycatch.me/asp-net-mvc-3-beta-razor-view-engine/razor/" rel="attachment wp-att-202"><img src="{{ site.baseurl }}/assets/razor-150x150.jpg" alt="Razor View Engine, Looking Sharp" title="Razor View Engine, Looking Sharp" width="150" height="150" class="size-thumbnail wp-image-202" /></a><br />
<a href="http://weblogs.asp.net/scottgu/archive/2010/10/06/announcing-nupack-asp-net-mvc-3-beta-and-webmatrix-beta-2.aspx">ASP.NET MVC 3 Beta</a> &amp; Enhancements to the Razor View Engine was announced by Scott Gutherie last week. </p>
<p>The Beta release also includes, New View Helpers, Unobtrusive JavaScript, Integration with the NuPack Package Manager and some other bells and whistles. So lets take a look inside and see what we can do.</p>
<p><!--more--></p>
<h3>Installation</h3>
<p>As usual, installation is pretty painless with both <a href="http://www.microsoft.com/web/gallery/install.aspx?appid=MVC3">Web Installer</a> &amp; <a href="http://go.microsoft.com/fwlink/?LinkID=191795">Manual Installer</a> available plus a <a href="http://www.asp.net/learn/whitepapers/mvc3-release-notes">myriad of documentation</a> about all that's new &amp; shiny in ASP.NET MVC 3</p>
<h3>Razor View-Engine</h3>
<p>If you've been following the Razor View Engine Development at all, you've probably seem some lovely images like these on ScottGu's Blog</p>
<p><a href="http://weblogs.asp.net/blogs/scottgu/image_thumb_39360205.png" title="Razor Syntax Highlighting, Example 1"><img src="{{ site.baseurl }}/assets/image_thumb_39360205.png" alt="Razor Syntax Highlighting" style="width: 40%;" /></a><a href="http://weblogs.asp.net/blogs/scottgu/image_thumb_3C0B1B1E.png" title="Razor Syntax Highlighting, Example 2"><img src="{{ site.baseurl }}/assets/image_thumb_3C0B1B1E.png" alt="Razor Syntax Highlighting" style="width: 40%;" /></a></p>
<p>Alas, the first thing that's blatantly apparent when you load up an Razor CSHtml file, is that there is no syntax highlighting or intellisense (or on-the-fly, as-you-type error checking) in this release. Apparently it's coming in a future release but it makes working with Razor a bit of a pain. In fact, Visual Studio 2010 got quite confused about syntax highlighting the CSHtml files and several times seemed to  apply plain-old-html highlightling on it instead of nothing at all. Some googling turns up that a few enterprising people have put together an <a href="http://visualstudiogallery.msdn.microsoft.com/en-us/8dc77b9c-7c83-4392-9c46-fd15f3927a2e">extension for highlighting</a></p>
<p>Razor syntax itself is nice, quick and intuitive to use. Want a new weakly-typed partial view, just add a .cshtml text file to your solution with the folllowing.</p>
<h3>Razor View Engine Syntax</h3>
```xml
@model dynamic
@using (Ajax.BeginForm("DummyMethod", "Home", 
    new AjaxOptions() { 
        UpdateTargetId = "MainContainer" 
    })) { 
    <input type="submit" value="Call Dummy Method" />
}
```

<h3>ASPX WebForms View Engine Syntax</h3>

```xml
<%@ Control Language="C#" Inherits="System.Web.Mvc.ViewUserControl<dynamic>" %@>
<%  using (Ajax.BeginForm("DummyMethod", "Home", 
       new AjaxOptions() { 
           UpdateTargetId = "MainContainer" 
       })) { %>
       <input type="submit" value="Call Dummy Method" />          
<% } %>
```
<p>After spending a few hours with it, it does make the code a little more readable & writeable, but then that's completely traded off against the lack of intellisense & syntax highlighting in this release.</p>
<h3>Unobtrusive Javascript</h3>
<p>This is a nice new feature of MVC 3. You can turn on unobtrusive javascript  on a per view basis or using an Application-wide configuration setting. <a href="http://en.wikipedia.org/wiki/Unobtrusive_JavaScript" title="Unobtrusive Javascript on Wikipedia">Unobtrusive Javascript</a> lets you keep a separation between your functional code ("behavior") and your structural markup & content ("presentation"). To give an example, the following snippet of Razor will generate an HTML Form with an AJAX Callback.</p>

```aspx-cs
@model dynamic
@using (Ajax.BeginForm("ChooseShow", "Home", 
	new AjaxOptions()
	{
		UpdateTargetId = "MainContainer",
		HttpMethod = "POST",
		OnSuccess = "ShowContentPanel",
		OnFailure = "ShowContentPanel",
		OnBegin = "ShowLoading"
	}))
{
	<input type="submit" value="Choose Show" />
}
```

<h3>Old School Rendering</h3>
<p>Previously the Microsoft & MVC Ajax Libraries would render JavaScript on your the &lt;form&gt; tag like this.</p>

```html
<form
    action="/Home/ChooseShow" 
    method="post" onclick="Sys.Mvc.AsyncForm.handleClick(this, new Sys.UI.DomEvent(event));" 
    onsubmit="Sys.Mvc.AsyncForm.handleSubmit(this, new Sys.UI.DomEvent(event), { 
        insertionMode: Sys.Mvc.InsertionMode.replace, 
        httpMethod: &#39;POST&#39;, 
        updateTargetId: &#39;MainContainer&#39;, 
        onBegin: Function.createDelegate(this, ShowLoading), 
        onFailure: Function.createDelegate(this, ShowContentPanel), 
        onSuccess: Function.createDelegate(this, ShowContentPanel) 
    });">
```

<p>So lets turn on Unobtrusive Javascript in the web.config</p>

```xml
<configuration>
  <appSettings>
   <add key="UnobtrusiveJavaScriptEnabled" value="true"/>
  </appSettings>
</configuration>
```

<h3>Unobstrusive Rendering</h3>

```html
<form action="/Home/ChooseShow" 
    data-ajax="true"
    data-ajax-begin="ShowLoading"
    data-ajax-failure="ShowContentPanel"
    data-ajax-method="POST"
    data-ajax-mode="replace"
    data-ajax-success="ShowContentPanel"
    data-ajax-update="#MainContainer"
    id="form0"
    method="post">
```

<p>The HTML footprint is much smaller than the previous. (Easier to read, smaller transmission size) and it's HTML 5 Compatible using <a href="http://dev.w3.org/html5/spec/Overview.html#embedding-custom-non-visible-data-with-the-data-attributes">Custom Data Attributes (data-*)</a></p>
<p>Brad Wilson has a bit more info on his blog post, <a href="http://bradwilson.typepad.com/blog/2010/10/mvc3-unobtrusive-ajax.html">Unobtrusive Ajax in ASP.NET MVC 3</a>, from earlier in the week.</p>
