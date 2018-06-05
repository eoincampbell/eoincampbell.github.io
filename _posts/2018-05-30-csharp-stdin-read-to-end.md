---
layout: post
title: Read to end using stdin from the console
description: 
tags: 
- console
- stdin
- standard input
- redirect
- read to end
categories: 
- c#
- .net
- coding
published: true
---

Your writing a console app and you want to continue to accept input from the user 
over multiple lines until they stop typing. Essentially "How do I do `ReadToEnd()` on the command line. 
Alternatively you want to be able to redirect input from another file.

Turns out it's quite easy to do.

```csharp
class Program
{
    static void Main(string[] args)
    {
        using (var sr = new StreamReader(Console.OpenStandardInput(), Console.InputEncoding))
        {
            var input = sr.ReadToEnd();
            var tokens = input.Replace(Environment.NewLine, " ").Split(' ');
            Console.WriteLine($"Tokens: {tokens.Count()}");
        }
    }
}
```

For the user interactive example you'll have to terminate the input with a CTRL-Z (or a CTRL-D on linux)

And you can now redirect/pipe to STDIN from other files.

```powershell
Get-Content .\input.txt | .\stdin-test.exe
``` 

***Eoin Campbell***