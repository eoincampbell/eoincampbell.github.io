---
layout: post
title: InstallUtil, Windows Services & ProjectInstallers with App.Config Settings
date: 2010-10-12 13:39:52.000000000 +01:00
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
- App.Config
- AppSetting
- Configuration
- InstallUtil
- ProjectInstaller
- Service
- Windows
meta:
  _edit_last: '1'
  _thumbnail_id: '233'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1525139002;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:714;}i:1;a:1:{s:2:"id";i:747;}i:2;a:1:{s:2:"id";i:462;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><a href="http://trycatch.me/installutil-windows-services-projectinstallers-with-app-config-settings/cogs/" rel="attachment wp-att-233"><img src="{{ site.baseurl }}/assets/cogs-150x150.jpg" alt="Config Fun" title="Config Fun" width="150" height="150" class="size-thumbnail wp-image-233" /></a>We had a situation in work where we needed to make service installation a more configurable process.</p>
<p>So a very simple example, In order to install a .NET Windows Service we need to provide it with a username & password that the services will run as. We can either provide that information at installation time, or through the following properties in the ProjectInstaller.cs file for your service.</p>
<p>However in an environment where multiple developers are working on a service, particularly a service that requires elevated privileges and needs to run as a specific account, this can be a royal pain. </p>
<p><!--more--></p>
```csharp
private void InitializeComponent()
{
    this.serviceProcessInstaller1 = new System.ServiceProcess.ServiceProcessInstaller();
    this.serviceInstaller1 = new System.ServiceProcess.ServiceInstaller();

    this.serviceProcessInstaller1.Username = "MYPC\JohnSmith";
    this.serviceProcessInstaller1.Password = "abcd1234";

    this.serviceInstaller1.ServiceName = "My Service";
    this.serviceInstaller1.Description = "My Description";

    this.Installers.AddRange(new System.Configuration.Install.Installer[] {
	this.serviceProcessInstaller1,	this.serviceInstaller1});
}
```
<p><strong>Option 1.</strong> Type in the credentials every time the service is Installed/Started... a not-so-ideal manual process when constantly re-starting a service to debug. It's also a pain for an automated xcopy releases where there is one big batch job which will stop, uninstall, reinstall & re-start every service in the deployment.</p>
<p><strong>Option 2.</strong> Leave the account credentials hard-coded in. This isn't ideal either. With Local Dev Environments, Shared Development Environments and Staging & Production platforms, it's too easy to leave the wrong credentials hardcoded into the ProjectInstaller, or worse, introduce a small typo/bug when the values are changed/recompiled between environment releases.</p>
<blockquote><p>Wouldn't it be better if this could be configurable ?</p></blockquote>
<p>Unfortunately, we need to do a little gymnastics in order to gain access to the .config file of the service at installation time when using InstallUtil.exe. The <em>App.config</em> Configuration File which the ConfigurationManager is using during the installation process is actually <em>InstallUtil.exe.config</em>... not what we want. Instead, we can manually load the associated configuration file for the assembly which contains the Project Installer, and retrieve our settings from that.</p>
```csharp
private static string GetConfigurationValue(string key)
{
    var service = Assembly.GetAssembly(typeof(ProjectInstaller));
    Configuration config = ConfigurationManager.OpenExeConfiguration(service.Location);
    if (config.AppSettings.Settings[key] == null)
    {
        throw new IndexOutOfRangeException("Settings collection does not contain the requested key:" + key);
    }

    return config.AppSettings.Settings[key].Value;
}
```
<p>We can now add our installation information to the configuration file which is template controlled under our release process. </p>
```csharp
private void InitializeComponent()
{
    this.serviceProcessInstaller1 = new System.ServiceProcess.ServiceProcessInstaller();
    this.serviceInstaller1 = new System.ServiceProcess.ServiceInstaller();

    this.serviceProcessInstaller1.Password = GetConfigurationValue("ServicePassword");
    this.serviceProcessInstaller1.Username = GetConfigurationValue("ServiceUserName");

    this.serviceInstaller1.ServiceName = "Service Name";
    this.serviceInstaller1.Description = "Service Description - " + GetConfigurationValue("ServiceLabel"); 
    //E.g. ServiceLabel = "LOCAL EOINC";

    this.Installers.AddRange(new System.Configuration.Install.Installer[] {
	this.serviceProcessInstaller1,
	this.serviceInstaller1});
}
```` 
<p>The usual caveats about config file encryptions and protecting your passwords apply.</p>

***Eoin Campbell***