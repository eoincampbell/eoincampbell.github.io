---
layout: post
title: System.Threading.Tasks & Parallelism in .NET 4.0
date: 2011-04-06 14:07:30.000000000 +01:00
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
- Multi-Threaded
- Paralell
- System.Threading
- Task
- Thread
meta:
  _edit_last: '1'
  _thumbnail_id: '441'
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><img class="size-medium wp-image-441" title="Parallel" src="{{ site.baseurl }}/assets/multi-300x165.jpg" alt="Parallel" width="300" height="165" /></p>
<p style="text-align: justify;">Alas, all my hope &amp; dreams &amp; promises of a regular blog post, dashed... oh well, here's one now.</p>
<p style="text-align: justify;">I've been playing with the <code><strong>System.Threading.Tasks</strong></code> namespace over the last few hours and it's quite neat.</p>
<p style="text-align: justify;">We'll be rolling out some new software in the next few months at work which Processes SMS messages from Customers. In the past we had fudged together our own Multi-Threading/Multi-Pipeline code to try and get messages through the system as quickly as possible but it was fairly bloated to say the least. Enter the new <code><strong>Task</strong></code> and <code><strong>Parallel</strong></code> classes in .NET 4.0</p>
<p><!--more--></p>
<p style="text-align: justify;">In our system, we take a collection of <code>MyObject</code>s from a DataSource. (usually DB or MSMQ). Each MyObject represents an instruction sent in by a customer so the time it takes to process a single MyObject can vary greatly depending on what part of the system the instruction needs to interact with.</p>
<p style="text-align: justify;">It doesn't make much sense from a UX point-of-view to process these in a linear sequential order waiting for one to complete before the next starts... e.g. if message A took 9 seconds to process &amp; messages B-J took one second each, you would ideally want to process B-J on a seperate thread and get them out in Paralell. Enter the <code><strong>Parallel.ForEach&lt;T&gt;</strong></code> &amp; <code><strong> Parallel.Invoke</strong></code> methods. These allow you to pass in a collection of Input Objects &amp; a delegate to process them, and the CLR will handle all the multi thread/multi core messiness for you under the hood.</p>
<p style="text-align: justify;">It comes with the usual abort handling code so that if you need to break out of the parallelism at some point (maybe due to some critical exception), you can. In our case this is necessary so we can requeue any messages that haven't yet been processed, if the Windows Service OnStop function is called.</p>

```csharp
public class MyObject
{
    public int MoID {get;set;}
    public bool ProcessedSuccessfully { get; set; }
    public MyObject(int moID)
    {
        MoID = moID;
        ProcessedSuccessfully = false;
    }
}

class Program
{
    public static void Main(string[] args)
    {
        //Using PARALELL
        //Simulate obtaining 50 objects from some datasource
        var inputs = Enumerable
            .Range(1, 50)
            .Select(i =&gt; new MyObject(i))
            .ToList();

        //Process Them in Paralell
        var po = new ParallelOptions() {
            MaxDegreeOfParallelism = 50
        };
        var pmR = Parallel.ForEach<MyObject>(inputs
            , po, ProcessMessage);

        //Once Processing is complete
        Console.WriteLine("Processing Done");

        //Check for any we didn't process / maybe due to internal
        //exceptions requesting that the processing stop
        // and do something with them, like requeue them for later.
        var fails = inputs
            .Where(i =&gt; !i.ProcessedSuccessfully);
        var bqR = Parallel.ForEach<MyObject>(fails, po, PutBackOnQueue);

        //Using TASK
        Console.ReadKey();
    }

    public static void ProcessMessage(MyObject message
        , ParallelLoopState pls, long l)
    {
        //Simulate a reason to stop on Message 25
        if (message.MoID < 25 && pls != null)
            pls.Break();

        Console.WriteLine("Starting ID: {0:0000}", message.MoID);

        Thread.Sleep(1000);

        message.ProcessedSuccessfully = true;
        Console.WriteLine("Stopping ID: {0:0000}", message.MoID);
    }

    public static void PutBackOnQueue(MyObject message)
    {
        Console.WriteLine("Queueing ID: {0:0000}", message.MoID);
    }
}
```

***Eoin Campbell***