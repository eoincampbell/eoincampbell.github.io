---
layout: post
title: Converting to Base62 & URL Shortening
date: 2010-10-04 21:32:52.000000000 +01:00
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
- Base10
- Shorten
- URL
meta:
  _edit_last: '1'
  _sexybookmarks_permaHash: 6a0d7206ee40ffcf1016276b14500a16
  _sexybookmarks_shortUrl: http://tinyurl.com/368vlpj
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1524924936;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:411;}i:1;a:1:{s:2:"id";i:876;}i:2;a:1:{s:2:"id";i:315;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
> *Generic Marketeer:* Why do we need to shorten the URLs, people just click on them. It's not like they have to memorize them; call them out to their friends
> *Me:* Eh... no you're right, but I do need to jam them into a SMS WAPPush along with some descriptive text, so I don't have much room to play with
> *Generic Marketeer:* Fine, as long as it doesn't impact the project deadline

Cue, my good self scurying away to find a way to bang out a private URL Shortening service in as short a time as possible. The URL Storage itself was a piece of pie. But I did stumble across some nice code while I was at it. I started out with a int-to-base64 implemenation. but the `+` & `/` characters are fugly to deal with in URLs. I tried a Hex version as well but it just didn't look bit-ly-y enough. Enter the baseAnything encoder. Just point it at any character set and it will encode/decode to that number of chars.
The following extension methods convert longs to strings, and vice-versa

``` csharp
public static string ToBase(this long input, string baseChars)
{
    string r = string.Empty;
    int targetBase = baseChars.Length;
    do
    {
        r = string.Format("{0}{1}",
            baseChars[(int)(input % targetBase)],
            r);
        input /= targetBase;
    } while (input &gt; 0);

    return r;
}

public static long FromBase(this string input, string baseChars)
{
    int srcBase = baseChars.Length;
    long id = 0;
    string r = input.Reverse();

    for (int i = 0; i &lt; r.Length; i++)
    {
        int charIndex = baseChars.IndexOf(r[i]);
        id += charIndex * (long)Math.Pow(srcBase, i);
    }

    return id;
}
```

I've been using it against the first character set below, but you could easily tweak it to remove any **confusing** characters, or any set for that matter.

``` csharp
private static string ALPHANUMERIC =
    "0123456789" +
    "ABCDEFGHIJKLMNOPQRSTUVWXYZ" +
    "abcdefghijklmnopqrstuvwxyz";

//Remove 0oO1iIl - Base52
private static string ALPHANUMERIC_ALT =
    "23456789" +
    "ABCDEFGHJKLMNPRSTUVWXYZ" +
    "abcdefghjkmnpqrstuvwxyz";
}
```

Now all I have to do is go write a Math Engine for it ;-)
