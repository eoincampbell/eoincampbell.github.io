---
layout: post
title: Combinatorics in .NET - Part I - Permutations, Combinations & Variations
date: 2012-09-01 22:28:03.000000000 +01:00
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
- ".NET C"
meta:
  _edit_last: '1'
  _thumbnail_id: '626'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1523857193;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:614;}i:1;a:1:{s:2:"id";i:802;}i:2;a:1:{s:2:"id";i:468;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><a href="http://trycatch.me/blog/wp-content/uploads/2012/09/combinatorics.png"><img class=" wp-image-626 " title="How many combinations?" src="{{ site.baseurl }}/assets/combinatorics.png" alt="How many combinations?" width="205" height="205" /></a> How many combinations?</p>
<p>[important]<br />
This is part 1 of a 2 part post on Combinatorics in .Net</p>
<p>The solution is publicly available on github; <a title="Combinatorics Solution on GitHub" href="https://github.com/eoincampbell/combinatorics" target="_blank">https://github.com/eoincampbell/combinatorics</a></p>
<p>The library can be added to any .NET Soution via Nuget; <a title="Combinatorics Package on Nuget" href="https://nuget.org/packages/Combinatorics" target="_blank">https://nuget.org/packages/Combinatorics</a><br />
[/important]</p>
<p>&nbsp;</p>
<p style="text-align: justify;">Recently while working on a project, I had need to generate combintations and permutations of sets of Inputs. In my search for a decent combinatorics library for .NET, (something which is missing from the BCL), I came across a <a title="Combinations, Permutations &amp; Variations in C#" href="http://www.codeproject.com/Articles/26050/Permutations-Combinations-and-Variations-using-C-G" target="_blank">Codeproject Article from Adrian Akision</a>. The implementation included a C# Generics Collection implementation for creating Combinations, Permutations &amp; Variations from an Input Set. Adrian very graciously allowed that I bundle up the code as a Class Library Solution on Github &amp; then release the library via Nuget.</p>
<h1 style="text-align: justify;"><!--more--></h1>
<h1 style="text-align: justify;">Permutations</h1>
<p>Permutations provide all possible ordering of an input set of items. For n elements, P(n) = n!</p>
<p><a href="http://trycatch.me/blog/wp-content/uploads/2012/09/perms_without_rep.png"><img class="wp-image-628 " title="Permutations" src="{{ site.baseurl }}/assets/perms_without_rep-300x152.png" alt="Permutations" width="240" height="122" /></a> Permutations</p>
<p>&nbsp;</p>
<p>e.g. How many ways can you shuffle the cards in a deck of playing cards.</p>
<p>&nbsp;</p>


### 3 Digit Permutations


```csharp
var integers = new List<int> {1, 2, 3};

var p = new Permutations<int>(integers);

foreach (var v in p)
{
    System.Diagnostics.Debug.WriteLine(string.Join(",", v));
}
```


```
Outputs:

1,2,3
1,3,2
2,1,3
2,3,1
3,1,2
3,2,1
```


<h1>Permutations with Repetition</h1>
<p style="text-align: justify;">Permutations with repetition take into account that some elements in the input set may repeat. In a 3 element input set, the number of permutations is 3! = 6. However if some of those input elements are repeated, then repeated output permutations would exist as well. Permutations with Repetition take account of repeating elements in the input set and do not disgard the repeated output sets. e.g.</p>
<p><a href="http://trycatch.me/blog/wp-content/uploads/2012/09/perms_with_rep.png"><img class=" wp-image-627 " title="Permutations with Repetition" src="{{ site.baseurl }}/assets/perms_with_rep-300x152.png" alt="Permutations with Repetition" width="240" height="122" /></a> Permutations with Repetition</p>
<p>&nbsp;</p>

## 3 Digit Permutations without repetition


```csharp
var integers = new List<int> {1, 1, 2};

var p = new Permutations<int>(integers, GenerateOption.WithoutRepetition);

foreach (var v in p)
{
    System.Diagnostics.Debug.WriteLine(string.Join(",", v));
}
```

```
Outputs:
1,1,2
1,2,1
2,1,1
```

## 3 Digit Permutations with repetition

```csharp
var integers = new List<int> {1, 1, 2};

var p = new Permutations<int>(integers, GenerateOption.WithRepetition);

foreach (var v in p)
{
    System.Diagnostics.Debug.WriteLine(string.Join(",", v));
}
```

```
Outputs:
1,1,2
1,2,1
1,1,2
1,2,1
2,1,1
2,1,1
```

<h1>Combinations</h1>
<p>Combinations are subsets of items taken from a larger set of items. E.g. How many five card hands can be drawn from a  deck of 52 cards. In a set of <em><strong>n </strong></em>items the total number of <em><strong>k</strong></em> sub-item combinations is calculated by <strong><em>n! / ( k! * (n - k)! )</em><em>.</em> </strong>In a deck of 52 cards, there are 2598960 combinations.</p>
<p><a href="http://trycatch.me/blog/wp-content/uploads/2012/09/cards.png"><img class="size-medium wp-image-633" title="52 Cards Choose 5" src="{{ site.baseurl }}/assets/cards-300x102.png" alt="52 Cards Choose 5" width="300" height="102" /></a> 52 Cards Choose 5</p>
<p>&nbsp;</p>

## 3 Digit Combinations from a 5 digit input set.

```csharp
var integers = new List {1, 2, 3, 4, 5};

var c = new Combinations(integers, 3);

foreach (var v in c)
{
    System.Diagnostics.Debug.WriteLine(string.Join(",", v));
}

Assert.AreEqual(10, c.Count);
```

```
Outputs:
1,2,3
1,2,4
1,2,5
1,3,4
1,3,5
1,4,5
2,3,4
2,3,5
2,4,5
3,4,5
```

<h1>Combinations with Repetition</h1>
<p>Combinations with repetition allow for repeating input values in the output subsets. This would be similar to the combinations you can generate from repeatedly rolling a dice.</p>
<p>&nbsp;</p>

## 2 Digit Combinations allowing Repetition from a set of 6


```csharp
var integers = new List { 1, 2, 3, 4, 5, 6};
var c = new Combinations(integers, 2, GenerateOption.WithRepetition);
foreach (var v in c)
{         
    System.Diagnostics.Debug.WriteLine(string.Join(",", v));
}
Assert.AreEqual(21, c.Count);
```

```
Outputs:
1,1  1,2  1,3
1,4  1,5  1,6
2,2  2,3  2,4
2,5  2,6  3,3
3,4  3,5  3,6
4,4  4,5  4,6
5,5  5,6  6,6
```

<h1>Variations</h1>
<p>Variations combine some of the properties of permutations &amp; combinations. They are the set of all ordered subsets of a particular input set. If the order the elements are selected is unimportant in a combination, then it is important in a variation.</p>
<p>&nbsp;</p>

```csharp
var integers = new List {1, 2, 3};

var v = new Variations(integers, 2);

foreach (var vv in v)
{
    System.Diagnostics.Debug.WriteLine(string.Join(",", vv));
}
```

```
Outputs:
1,2
1,3
2,1
3,1
2,3
3,2
```

# Variations with Repetition

## 2 Digit Variations from a 3 Digit Input Set with repetition allowed

```csharp
var integers = new List { 1, 2, 3};

var v = new Variations(integers, 2, GenerateOption.WithRepetition);

foreach (var vv in v)
{
    System.Diagnostics.Debug.WriteLine(string.Join(",", vv));
}
```

```
Outputs:
1,1
1,2
1,3
2,1
2,2
2,3
3,1
3,2
3,3
```

<p>In the next post we'll look at the organisation of the solution and how it was deployed to <a title="Nuget.org" href="http://nuget.org" target="_blank">Nuget.org</a></p>
<p><em>~Eoin Campbell</em></p>
