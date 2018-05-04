---
layout: post
title: LINQ Deferred Execution & Lambda Methods for providing Simple Stats (Part II)
date: 2013-05-24 11:18:01.000000000 +01:00
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
- any query
- average
- count
- deferred execution
- execution
- IEnumerable
- lamdba
- linq
- max
- min
meta:
  _edit_last: '1'
  _wpas_done_all: '1'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1522894575;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:867;}i:1;a:1:{s:2:"id";i:369;}i:2;a:1:{s:2:"id";i:242;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p>[su_note note_color="#dee3ab" text_color="#5E826F"]<span style="font-size: medium;">This is part 2 in a series of posts on <a title="LINQ &amp; Lambda Series" href="http://trycatch.me/category/coding/linq-lambda-series/" target="_blank">Linq &amp; Lambda capabilities in C#</a> </span>[/su_note]</p>
<h2>Deferred Execution</h2>
<p style="text-align: justify;">So lets take a minute to talk about deferred execution. You may here this referred to as Lazy Execution as well. But in a nutshell what this means is that when you write a linq or lambda query against a collection or list, the execution of that query doesn't actually happen until the point where you need to access the resuts. Let's look at a simple example.</p>
```csharp
var ienum = Enumerable.Range(1, 10).ToList();

var query = from i in ienum
            where i%2 == 0
            select i;

ienum.Add(20);
ienum.Add(30);

SuperConsole.WriteLine(query);
//prints 2, 4, 6, 8, 10, 20, 30
```
<p style="text-align: justify;">So why does it print out 20 and 30. This is deferred execution in practice. At the point where you write your query (<span style="font-family: 'courier new', courier;"><strong>var query</strong></span>) the query is not actually executed against your datasource (<span style="font-family: 'courier new', courier;"><strong>ienum</strong></span>). After the query is setup, more data is added to your data source, and the query is only actually executed at the point where the results need to be evaluated (<span style="font-family: 'courier new', courier;"><strong>SuperConsole.WriteLine</strong></span>)</p>
<p style="text-align: justify;">This holds true in a number of other Linq Scenarios. In Linq-to-Sql or Linq-to-Entity Framework, execution of the Sql Query is only sent to the database at the point where you need to evaluate your queries. It's important to understand this so that queries don't go out of scope before being executed, so that un-executed queries aren't inadvertently passed to other parts or layers in your application and so that you don't end up introducing N+1 problems where you think your working on data in memory but in actual fact, your performing multiple executions over and over in a loop. If you do need to make your queries "Greedy" and force them to execute there and then, you can wrap them in parenthesis and immediately call<span style="font-family: 'courier new', courier;"><strong> .ToList()</strong> </span>on them to force the execution.</p>
<h2>Min, Max, Count &amp; Average</h2>
<p style="text-align: justify;">Linq has a number of convenient built in methods for getting various numeric stats about the data your working on. Consider a collection of Movies which you want to Query.</p>
```csharp
public class Movie
{
    public string Title { get; set; }
    public double Rating { get; set; }
}

...

var movies = new List
    {
        new Movie() {Title = "Die Hard", Rating = 4.0},
        new Movie() {Title = "Commando", Rating = 5.0},
        new Movie() {Title = "Matrix Revolutions", Rating = 2.1}
    };

Console.WriteLine(movies.Min(m =&gt; m.Rating));
//prints 2.1

Console.WriteLine(movies.Max(m =&gt; m.Rating));
//prints 5

Console.WriteLine(movies.Average(m =&gt; m.Rating));
//prints 3.7

Console.WriteLine(movies.Count);
Console.WriteLine(movies.Count());
//prints 3
```
<p style="text-align: justify;">Min, Max and Average are all fairly straight forward, finding the Minimum, Maximum and Average movie rating values respectively. It's worth mentioning with regards the Count implementations that there are different "versions" of the count implementation depending on the underlying data structure you are operating on. The Count property is a property of the List class are returns the current number of items in that collection. The Count() method is an extension method on the IEnumerable interface which can be executed on any IEnumerable structure regardless of implementation.</p>
<blockquote><p>In general LINQ's Count will be slower and is an O(N) operation while List.Count and Array.Length are both guaranteed to be O(1). However in some cases LINQ will special case the IEnumerable parameter by casting to certain interface types such as IList or ICollection. It will then use that Count method to do an actual Count() operation. So it will go back down to O(1). But you still pay the minor overhead of the cast and interface call. Ref: [<a href="http://stackoverflow.com/questions/981254/is-the-linq-count-faster-or-slower-than-list-count-or-array-length/981283#981283">http://stackoverflow.com/questions/981254/is-the-linq-count-faster-or-slower-than-list-count-or-array-length/981283#981283</a>]</p></blockquote>
<p style="text-align: justify;">This is important as well if you are testing your collections to see if they are empty. People coming from versions of .NET previous to Generics would use the Count or Length properties of a collection to see if they were empty. i.e.</p>
```csharp
if(list.Count == 0)
{ 
    //empty
}
if(array.Length == 0)
{
    //empty
}
```
<p>Linq however provides another method to test for contents called Any(). It can be used to evaluate whether the collection is empty, or if the collection has any items which validate a specific filter.</p>
```csharp
if(list.Any()) //equivalent of count == 0
{ 
    //empty
}
if(list.Any(m => m.Rating == 5.0)) //if it contains any top rated movies.
{
    //empty
}
```
<blockquote><p>If you are starting with something that has a .Length or .Count (such as ICollection, IList, List, etc) - then this will be the fastest option, since it doesn't need to go through the GetEnumerator()/MoveNext()/Dispose() sequence required by Any() to check for a non-empty IEnumerable sequence. For just IEnumerable, then Any() will generally be quicker, as it only has to look at one iteration. However, note that the LINQ-to-Objects implementation of Count() does check for ICollection (using .Count as an optimisation) - so if your underlying data-source is directly a list/collection, there won't be a huge difference. Don't ask me why it doesn't use the non-generic ICollection... Of course, if you have used LINQ to filter it etc (Where etc), you will have an iterator-block based sequence, and so this ICollection optimisation is useless. In general with IEnumerable : stick with Any() Ref: [<a href="http://stackoverflow.com/questions/305092/which-method-performs-better-any-vs-count-0/305156#305156">http://stackoverflow.com/questions/305092/which-method-performs-better-any-vs-count-0/305156#305156</a>]</p></blockquote>
<p>Next post, we'll look at some different mechanisms for filtering and transforming our queries.</p>
<p><em> ~Eoin C</em></p>
