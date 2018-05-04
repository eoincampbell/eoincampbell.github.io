---
layout: post
title: Unexpected Variable Behaviour in DOS Batch and Delayed Expansion
date: 2012-12-04 15:07:12.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Coding
- Random
tags:
- batch
- batch command
- delayed expansion
- dos
- enabledelayedexpansion
- script
- variable
meta:
  _edit_last: '1'
  _wpas_done_all: '1'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1524935815;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:468;}i:1;a:1:{s:2:"id";i:614;}i:2;a:1:{s:2:"id";i:835;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p style="text-align: justify;">What would you expect the following piece of Code to print. if the directory 'A' doesn't exist</p>
<pre class="brush:text;">@ECHO OFF
IF '1'=='1' (
        CD a
        ECHO %ERRORLEVEL%
)

CD a
ECHO %ERRORLEVEL%</pre>
<div></div>
<div><a href="http://trycatch.me/blog/wp-content/uploads/2012/12/cmd.jpg"><img class=" wp-image-739 aligncenter" title="cmd" src="{{ site.baseurl }}/assets/cmd.jpg" alt="" width="543" height="227" /></a></div>
<div></div>
<h1 style="text-align: justify;">Not very intuitive right?</h1>
<div style="text-align: justify;"></div>
<div style="text-align: justify;">This is because the DOS batch processor treats the whole if statement as one command, expanding the variables only once, before it executes the conditional block. So you end up with %ERRORLEVEL% being expanded to its value, which is 0, before you start the block, You can get around this by enabling Delayed Expansion. As the name suggests this forces the the Batch Processor to only expand variables once required to do so in the middle of execution.</div>
<div style="text-align: justify;"></div>
<div style="text-align: justify;">To enable this behavior you need to do 2 things.</div>
<div>
<ol style="text-align: justify;">
<li>SET ENABLEDELAYEDEXPANSION at the top of your script.</li>
<li>Replace % delimited variables with an Exclamation. i.e. <strong>%ERRORLEVEL%</strong> becomes<strong> !ERRORLEVEL!</strong></li>
</ol>
<p style="text-align: justify;">Now our script looks like this, and behaves as expected.</p>
<h1 style="text-align: justify;">Working Script</h1>
</div>
<pre class="brush:text;">@ECHO OFF
REM Enable Delayed Expansion
setlocal enabledelayedexpansion
IF '1'=='1' (
        CD a
        REM Use Exclamations instead of percentages
        ECHO !ERRORLEVEL!
)

CD a
ECHO %ERRORLEVEL%</pre>
<div>For when powershell just isn't retro enough ;-)<br />
<em>~Eoin C</em></div>
