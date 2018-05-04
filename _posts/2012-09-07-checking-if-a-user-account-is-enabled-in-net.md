---
layout: post
title: Checking if a user account is enabled in .NET
date: 2012-09-07 16:27:51.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- C#
- Coding
- Random
tags:
- account
- c#. .net
- directory
- directoryentry
- directoryservices
- disabled
- enabled
- ntacount
- user
- Windows
meta:
  _edit_last: '1'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1525236826;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:226;}i:1;a:1:{s:2:"id";i:747;}i:2;a:1:{s:2:"id";i:714;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p>Here's a handy little code snippet to figure out if a local windows user account is enabled or not using the <code>System.DirectoryServices</code> namespace.</p>

```csharp
private static bool IsUserAccountEnabled(string username)
{
    try
    {
        var result = new DirectoryEntry { Path = "WinNT://" + Environment.MachineName + ",computer" }
            .Children
            .Cast<DirectoryEntry>()
            .Where(d => d.SchemaClassName == "User")
            .First(d => d.Properties["Name"].Value.ToString() == username);

        return ((int)result.Properties["UserFlags"].Value & 2) != 2;
    }
    catch
    {
        return false;
    }
}
```

<p><em>~Eoin Campbell</em></p>
