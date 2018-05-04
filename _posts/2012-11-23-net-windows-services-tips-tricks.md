---
layout: post
title: ".NET Windows Services - Tips & Tricks"
date: 2012-11-23 17:30:34.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- Coding
- Visual Studio
tags:
- ".net"
- c#
- debug
- debugger
- debugging
- F5
- install
- InstallUtil
- self-install
- Service
- Visual Studio
- windows service
meta:
  _syntaxhighlighter_encoded: '1'
  _wpas_skip_2206960: '1'
  _wpas_skip_2206956: '1'
  _edit_last: '1'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1524559183;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:226;}i:1;a:1:{s:2:"id";i:818;}i:2;a:1:{s:2:"id";i:614;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p>[caption id="" align="alignright" width="179"]<img class=" " title="Windows Services" src="{{ site.baseurl }}/assets/services.png" alt="Windows Services" width="179" height="179" /> Windows Services[/caption]</p>
<p style="text-align: justify;">If you've ever worked with windows services, you'll know that they're a very powerful tool to have in your background processing arsenal. Unfortunately they can be also quite a pain to work with in developer land. Recently we've been spinning up a lot of new windows service projects in work as part of a Business Intelligence Data Processing Project. I thought this would be a good time to brain dump some of the tips &amp; tricks I've come across over the past few years for dealing with .Net Windows Services.</p>
<p style="text-align: justify;">I'll look at the basics for getting a service up and going, using the built project installer &amp; Install Util. Then I'll take a look at easier ways of running the service inside the IDE, and how to run the service in user interactive mode.</p>
<p style="text-align: justify;">Finally I'll look at ways to make the service self-installing without having to rely upon the InstallUtil.exe as well as gaining access to configuration settings during the installation process.</p>
<p style="text-align: justify;">[important]The completed solution can be found on GitHub at <a href="https://github.com/eoincampbell/demo-windows-service">https://github.com/eoincampbell/demo-windows-service</a> [/important]</p>
<p style="text-align: justify;"><!--more--></p>
<h1 style="text-align: justify;">Windows Services Basics</h1>
<p style="text-align: justify;">I'm going to build a very trivial Windows Service. The service will start a timer, which will then log the current time every 100ms to a file in the same path  as the executable. It will also attempt to write to the console if there is one attached. A few notes. Don't leave this running. It will devour your disk space. You'll also need to make sure that the account you install the service as has permission to write to that path on the file system.</p>
<p style="text-align: justify;">When I first create a new windows service, there are a number of <em><strong>setup</strong></em> tasks I perform to keep the service organised. YMMV with these.</p>
<ol>
<li>If I'm only adding a single Service.cs in this project, then I'll rename the service class to the same name as the project. This will make the <em>Service Name</em> match with the <em>Executable Name</em> which I've found saves confusion. In my example the Project, Service Class, Namespace &amp; Executable are all <em><strong>DemoWindowsService.</strong></em></li>
<li>Open the Service.Designer.cs file and ensure that the property <em>this.ServiceName  </em>also matches this naming convention</li>
<li>Add an installer to your service by Right-Clicking the design canvas of the service and choosing "Add Installer" (See Image Below)</li>
<li>Once you've added your ProjectInstaller open that file also and ensure that the property <em>this.serviceInstaller1.ServiceName </em>also matches this naming convention</li>
<li>We'll also  specify a Description on the serviceInstaller1 object to give our service a meaningful description when it appears in services.msc listing</li>
</ol>
<p>[caption id="attachment_716" align="aligncenter" width="300"]<a href="http://trycatch.me/blog/wp-content/uploads/2012/11/1.png"><img class="size-medium wp-image-716" title="Add Installer to Windows Service" src="{{ site.baseurl }}/assets/1-300x226.png" alt="Add Installer to Windows Service" width="300" height="226" /></a> Add Installer to Windows Service[/caption]</p>
<p style="text-align: justify;">After building your windows service application, you can then use InstallUtil.exe from the command line to install or uninstall your applicaiton.</p>
<p>[text]InstallUtil.exe DemoWindowsService.exe<br />
net start DemoWindowsService<br />
net stop DemoWindowsService<br />
InstallUtil.exe /u DemoWindowsService.exe[/text]</p>
<h1 style="text-align: justify;">Debugging Windows Services</h1>
<p style="text-align: justify;">Relying on InstallUtil is all well and good but it doesn't lend itself to an easy developer debugging experience. If you attempt to press F5 in order to start your Windows Service from within the IDE, you'll be presented with the following rather unhelpful popup.</p>
<p>[caption id="attachment_717" align="aligncenter" width="533"]<a href="http://trycatch.me/blog/wp-content/uploads/2012/11/2.png"><img class="size-full wp-image-717" title="Windows Service Start Failure" src="{{ site.baseurl }}/assets/2.png" alt="Windows Service Start Failure" width="533" height="237" /></a> Windows Service Start Failure[/caption]</p>
<p style="text-align: justify;">This is because the default code which is added to program.cs relies on launching the Service using the static ServiceBase.Run() method. This is the entry point for <em><strong>"the scum"</strong></em>. (The SCM, or Service Control Manager is the external program which controls launching the services). We can still debug our Application from here but it's a little bit round about.</p>
<ol>
<li>Build the Service.</li>
<li>install it using InstallUtil.exe</li>
<li>NET START the service.</li>
<li>Attach to that Process using the Debug menu in Visual Studio.</li>
<li>Find a bug.</li>
<li>Detachn &amp; NET STOP the service.</li>
<li>Uninstall, re-build, wash-rinse-repeat.</li>
</ol>
<h1>Using Debugger.Launch &amp; Debugger.Break</h1>
<p style="text-align: justify;">Another option available to trigger debugging is to embed calls to the Debugger directly in your code. These calls could be surrounded by <em>Conditional</em> attributes to prevent Debug Launch statements leaking into release versions of code.</p>
```csharp
protected override void OnStart(string[] args)
        {
            DebugLaunch();
            MainTimer.Enabled = true;
        }

        private void MainTimer_Elapsed(object sender, System.Timers.ElapsedEventArgs e)
        {
            DebugBreak();
            var path = Assembly.GetExecutingAssembly().Location;
            var dir = Path.GetDirectoryName(path);
            var text = string.Format("Tick: {0:yyyy-MM-dd HH:mm:ss.fff}{1}", DateTime.Now, Environment.NewLine);
            File.AppendAllText(dir + "\output.log", text);
            Console.Write(text);
        }

        [Conditional("DEBUG")]
        public void DebugLaunch()
        {
            Debugger.Launch();
        }

        [Conditional("DEBUG")]
        public void DebugBreak()
        {
            Debugger.Break();
        }
```

<p style="text-align: justify;">[notice]Unfortunately it would appear that this doesn't work in Windows 8. Microsoft have slowly been phasing out the ability of Windows Services to run in Interactive mode and interact with the desktop. Since the Debugger.Launch statement needs to load a GUI for the user to interact with, the Windows Service would appear to hang on this statement. [/notice]</p>
<h1 style="text-align: justify;">Launch Windows Service with F5</h1>
<p style="text-align: justify;">What would be very helpful is if we could just launch our application from within the Debugger as needed. Well it turns out we can by conditionally launching the service using the ServiceBase.Run() method or by just launching it as a regular instantiated class.</p>
<pre class="brush:csharp;highlight: [7];">    static class Program
    {
        static void Main(string [] args)
        {
            var service = new DemoService();

            if (Debugger.IsAttached)
            {
                service.InteractiveStart(args);
                Console.WriteLine("Press any key to stop!");
                Console.Read();
                service.InteractiveStop();
            }
            else
            {
                ServiceBase.Run(service);
            }
        }
    }</pre>
<p style="text-align: justify;">Now if I press F5, the application entry point will check if the application is in Debug Mode (Debugger Attached) and if so will simply launch the application as a console application. In order to get this to work I've had to make a few small changes.</p>
<ol>
<li>Go to the Project Properties screen &amp; change the application type from "Windows Application" to "Console Application"</li>
<li>Add two public wrapper methods to the service for InteractiveStart &amp; InteractiveStop since the OnStart &amp; OnStop methods are protected</li>
<li>Add a Console.Read() between calling Start and Stop to prevent the Service from immediately shutting down.</li>
</ol>
<div></div>
<h1>Command Line Switch Driven Behaviour</h1>
<p style="text-align: justify;">Relying on the Debugger being attached is all well and good but what if I just want to run my application optionally in stand-alone mode. We could extend the runtime condition from the last code snippet to also test whether the application is running in InteractiveMode. But I think I'd prefer a little more power so I've added a command line switch to create this behavior. Now I have the option to Install from the command line using InstallUtil, run from the command line interactively, and I can set a startup switch argument in Project Properties -&gt; Debug -&gt; Start Options to run the application in Console mode from the IDE.</p>
<pre class="brush:csharp;highlight: [7];">    static class Program
    {
        static void Main(string [] args)
        {
            var service = new DemoService();

            if (args.Any() &amp;&amp; args[0].ToLowerInvariant() == "--console")
            {
                RunInteractive(service,args);
            }
            else
            {
                ServiceBase.Run(service);
            }
        }

        private static void RunInteractive(DemoService service, string [] args)
        {
            service.InteractiveStart(args);
            Console.WriteLine("Press any key to stop!");
            Console.Read();
            service.InteractiveStop();
        }
    }</pre>
<h1 style="text-align: justify;">Self Managed Installation</h1>
<p style="text-align: justify;">My service is becoming more self-sufficient and useful but I still have this external dependency on InstallUtil. Wouldn't it be great if I could just drop my application on a server and have it install itself. Enter the <em>ManagedInstallerClass</em></p>
<pre class="brush:csharp;highlight: [11,14];">    static void Main(string [] args)
    {
        var service = new DemoService();
        var arguments = string.Concat(args);
        switch(arguments)
        {
            case "--console":
                RunInteractive(service,args);
                break;
            case "--install":
                ManagedInstallerClass.InstallHelper(new [] { Assembly.GetExecutingAssembly().Location });
                break;
            case "--uninstall":
                ManagedInstallerClass.InstallHelper(new [] { "/u", Assembly.GetExecutingAssembly().Location });
                break;
            default:
                ServiceBase.Run(service);
                break;
        }
    }</pre>
<p style="text-align: justify;">If you want to simplify matters even further, you can modify your ProjectInstaller.Designer.cs file and specify that the Service should be of type AutomaticStart</p>
<pre class="brush:csharp;highlight: [5];">    // 
    // serviceInstaller1
    // 
    this.serviceInstaller1.ServiceName = "DemoWindowsService";
    this.serviceInstaller1.StartType = ServiceStartMode.Automatic;</pre>
<h1 style="text-align: justify;">InstallUtil Install-Time Configuration</h1>
<p style="text-align: justify;">One final thing you might find useful is the ability to query values from your configuration file when using Intsall Util. The problem here is that the AppSettings collection during installation is not the collection in the service.config but the collection in the InstallUtil app.config. Have a read of the following link for getting access to your own config at install time (e.g. in order to install with Default Supplied UserName &amp; Password).</p>
<p style="text-align: justify;"><a href="http://trycatch.me/installutil-windows-services-projectinstallers-with-app-config-settings/">http://trycatch.me/installutil-windows-services-projectinstallers-with-app-config-settings/</a></p>
<h1>Future Proofing</h1>
<p style="text-align: justify;">One last piece of advice. If you're building any sort of sizeable windows service/back-end processor, I would strongly suggest you <em><strong>NOT </strong></em>implement any logic in the service class itself. Typically I will create some sort of "Engine" class that encapsulates all my functionality with 3 external accessible members.</p>
<ul style="text-align: justify;">
<li>A public constructor called from Service Constructor</li>
<li>A public Start or Init method, called from OnStart()</li>
<li>A public Stop or End method, called from OnStop()</li>
</ul>
<p style="text-align: justify;">The reason for this is simply to prevent lock-in &amp; tight coupling to the windows service treating it only as a helpful host. This allows me to easily spin up my Engine class in a standalone Console Application, inside a web apps Global.asax or in an Azure Worker role with a minimum of refactoring in order to extract the functionality.</p>
<p><em>~Eoin C</em></p>
