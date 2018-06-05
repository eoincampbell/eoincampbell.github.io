---
layout: post
title: Using Signlar to Publish Dashboard Data
date: 2013-05-15 15:09:08.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ASP.NET
- C#
- Coding
tags:
- ASP.NET
- client
- client-side
- dashboard
- hub
- Javascript
- live
- live updates
- operations
- processor
- realtime
- signalr
meta:
  _edit_last: '1'
  _wpas_done_all: '1'
  _wpas_skip_2206960: '1'
  _wpas_skip_2206956: '1'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1522954892;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:614;}i:1;a:1:{s:2:"id";i:160;}i:2;a:1:{s:2:"id";i:714;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><img class=" wp-image-836  " title="SignalR" alt="SignalR" src="{{ site.baseurl }}/assets/signalr.png" width="151" height="151" /> SignalR</p>
<p>Recently <a title="David Fowler @ Twitter" href="https://twitter.com/davidfowl/" target="_blank">David </a><a title="David Fowler @ MSDN" href="http://weblogs.asp.net/davidfowler/" target="_blank">Fowler </a>announced the release of the Signlar 1.1.0 Beta Release. So I decided to do some dabbling to get a prototype application up and running. The solution is pretty simple. It uses a SignlaR hub to broadcast the current Processor % usage, and renders it in a nice visual graph using <a title="Highcharts interactive Javascript Charts" href="http://www.highcharts.com/" target="_blank">HighCharts</a>.</p>
<p>&nbsp;</p>
<div>[important]The completed solution can be found on GitHub at <a href="https://github.com/eoincampbell/signalr-processor-demo">https://github.com/eoincampbell/signalr-processor-demo</a> [/important]</div>
<p>First things first we'll need a bare bones web application which we can pull in the relevant nuget packages into. I started with a basic empty web application running under .NET 4.5. Installing the signlar &amp; highcharts packages is a breeze. Open up the PowerShell Nuget Console and run the following commands. HighCharts gets installed as a solution level package so you'll need to manually copy the relevant JavaScript files to your scripts directory in your application.</p>

```
Install-Package HighCharts
Install-Package Microsoft.AspNet.SignalR
```

<h2>The Hub</h2>
<p>Signalr relies on a "Hub" to push data back to all the connected Clients. I've created a "ProcessorDataHub" which implements the Signalr Base Hub to manage this process. It contains a constructor for Initializing a static instance of my ProcessorTicker class, and a start method to start the thread within the ticker. The HubName attribute specifies the name which the hub will be accessible by on the Javascript side.</p>

```csharp
[HubName("processorTicker")]
public class ProcessorDataHub : Hub
{
    private readonly ProcessorTicker _ticker;

    public ProcessorDataHub() : this(ProcessorTicker.Instance) { }

    public ProcessorDataHub(ProcessorTicker ticker)
    {
        _ticker = ticker;
    }

    public void Start()
    {
        _ticker.Start(Clients);
    }
}</pre>
<h2>The ProcessorTicker</h2>
<p>The heavy lifting is then done by the ProcessorTicker. This is instantiated with a reference to the Clients object, a HubConnectionContext which contains dynamic objects allowing you to push notifications to some or all connected client side callers. The implementation is fairlly simple using a System.Thread.Timer which reads the current processor level from a peformance counter once per second, and Broadcasts that value to the client side.</p>
<p>Since the Clients.All connection is dynamic, calling "updateCpuUsage" on this object will work at runtime, so long as the relevant client side wiring up to that expected method has been done correctly.</p>
```csharpClients.All.updateCpuUsage(percentage);</pre>
<h2>The Client Side</h2>
<p>One change since the previous version of SignalR is the requirement for the developer to manually &amp; explicity wireup the dynamically generated Javascript endpoint where SignalR creates it's javascript. This can be done on Application Start by calling the RouteTable..Routes.MapHubs() method</p>

```csharp
protected void Application_Start(object sender, EventArgs e)
{
RouteTable.Routes.MapHubs();
}
```
<p>Finally we're ready to consume these published messages on our Client Page. Signlar requires the following javascript includes in the Head Section of your page.</p>

```xml
<script type="text/javascript" src="/Signalr/Scripts/jquery-1.6.4.js"></script>
<script type="text/javascript" src="/Signalr/Scripts/jquery.signalR-1.1.0-beta1.js"></script>
<script type="text/javascript" src="/Signalr/signalr/hubs"></script>
```

<p>With those inplace, we wire up our own custom Javascript function to access our ProcessorTicker, start the Hub on a button click, and begin receiving and processing the</p>

```xml
<script type="text/javascript">
    $(function () {
        var ticker = $.connection.processorTicker;

        //HighCharts JS Omitted..

        ticker.client.updateCpuUsage = function (percentage) {
            $("#processorTicker").text("" + percentage + "%");

            var x = (new Date()).getTime(), // current time
                y = percentage,
                series = chart.series[0];

            series.addPoint([x, y], true, true);
        };

        // Start the connection
        $.connection.hub.start(function () {
            //alert('Started');
        });

        // Wire up the buttons
        $("#start").click(function () {
            ticker.server.start();
        });
    });
</script>
``` 

<p>The result is that I can fire up a number of separate browser instances and they'll all get the correct values published to them from the hub over a persistent long running response. Obviously this an extremely powerful system that could be applied to Live Operations Systems where dash boards have traditionally relied on polling the server at some regular interval.</p>
<p><img class="size-large wp-image-839" alt="Live Processor Data to Multiple Browsers via SignalR" src="{{ site.baseurl }}/assets/Processor-1024x559.png" width="840" height="458" />Live Processor Data to Multiple Browsers via SignalR</p>
<p><em>~Eoin Campbell</em></p>
