---
layout: post
title: Handy LINQ & Lambda Methods and Extensions (Part I)
date: 2013-05-23 17:15:11.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- C#
- Coding
- LINQ &amp; Lambda Series
tags:
- Extension Methods
- IEnumerable
- lambda
- linq
- Toolbelt
- Tools
- Utilities
meta:
  _edit_last: '1'
  _wpas_done_all: '1'
  _wpas_skip_2206960: '1'
  _wpas_skip_2206956: '1'
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p>The <span style="font-family: 'courier new', courier;"><strong>System.Linq</strong> </span>namespace contains a fantastic set of utility extension methods for filtering, ordering &amp; manipulating the contents of your collections and objects. In the following posts I'll go through some of the most useful ones (in my humble opinion) and how you might use them in your C# solutions<br />
[su_note note_color="#dee3ab" text_color="#5E826F"]<span style="font-size: medium;">This is part 1 in a series of posts on <a title="LINQ &amp; Lambda Series" href="http://trycatch.me/category/coding/linq-lambda-series/" target="_blank">Linq &amp; Lambda capabilities in C#</a> </span>[/su_note]</p>
<p>Before we start, here's a handy static method to print your resulting collections to the console so you can quickly verify the results.</p>
<pre class="brush:csharp;">public class SuperConsole
{
    public static void WriteLine&lt;T&gt;(IEnumerable&lt;T&gt; list, bool includeCarriageReturnBetweenItems =false)
    {
        var seperator = includeCarriageReturnBetweenItems ? ",\n" : ", ";
        var result = string.Join(seperator, list);
        Console.WriteLine(result);
    }
}</pre>
<h2>Enumerable</h2>
<p>The<span style="font-family: 'courier new', courier;"><strong> System.Linq.Enumerable</strong></span> type has 2 very useful static methods on it for quickly generating a sequence of items. <strong><span style="font-family: 'courier new', courier;">Enumerable.Range</span></strong> &amp; <strong><span style="font-family: 'courier new', courier;">Enumerable.Repeat</span></strong>. The Range method allows you to quickly generate a sequential list of integers from a given starting point for a given number of items.</p>
<pre class="brush:csharp;">IEnumerable&lt;int&gt; range = Enumerable.Range(1, 10);
SuperConsole.WriteLine(range);
//prints "1, 2, 3, 4, 5, 6, 7, 8, 9, 10"</pre>
<p>So why is this useful, well you could use it to quickly generate a pre-initialised list of integers rather than new'ing up a list and then iterating over it to populate it. Or you could use it to replicate <span style="font-family: 'courier new', courier;"><b>for(;;)</b> </span>behavior. e.g.</p>
<pre class="brush:csharp;">for (int i = 1; i &lt;= 10; i++) 
{     
    //DoWork(i); 
} 

Enumerable.Range(1, 10).ToList().ForEach(i =&gt;
    {
        //DoWork(i)
    });</pre>
<p>Repeat is similar but is not limited to integers. You can generate a Sequence of a given length with the same default value in every item. Imagine you wanted to create a list of 10 strings all initialised with a default string of "ABC";</p>
<pre class="brush:csharp;">var myList = Enumerable.Repeat("ABC", 10).ToList();</pre>
<h2>Item Conversion</h2>
<p>There are also a few handy ways to convert/cast items built into the System.Linq namespace. The <span style="font-family: 'courier new', courier;"><strong>Cast&lt;T&gt;</strong></span> extension method allows you to cast a list of variables from one type to another as long as a valid cast is available. This can be useful for quickly changing a collection of super types into their base types.</p>
<pre class="brush:csharp;">var integers = Enumerable.Range(1, 5);
var objects = integers.Cast&lt;object&gt;().ToList();

Console.WriteLine(objects.GetType());
SuperConsole.WriteLine(objects);

//prints
//System.Collections.Generic.List`1[System.Object]
//1, 2, 3, 4, 5</pre>
<p>But what if a valid implicit cast isn't available. What if we wanted to convert our collection of integers into a collection of strings with a ':' suffix. Thankfully Linq has us covered with it's ConvertAll Method on List</p>
<pre class="brush:csharp;">var integers = Enumerable.Range(1, 5);
var converter = new Converter&lt;int, string&gt;(input =&gt; string.Format("{0}: ", input));
var results = integers.ToList().ConvertAll(converter);

SuperConsole.WriteLine(results, true);
/*prints
    1:
    2:
    3:
    4:
    5:
    */</pre>
<p>In the next post, we'll look at some the lazy &amp; deferred execution capabilities of LINQ and some useful methods for performing quick calculations and manipulations on our collections.</p>
<p><em>~Eoin C</em></p>
