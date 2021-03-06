---
layout: post
title: Running SQL Server on Azure Virtual Machines
date: 2016-01-31 19:00:49.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Environments &amp; Tools
- Windows Azure
tags:
- Azure
- Cloud
- MSSQL
- SQL
- sql server
- Virtual Machine
- VM
meta:
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1525303673;s:7:"payload";a:0:{}}}
  _edit_last: '1'
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<h2>Introduction</h2>
<p>We recently started hitting some capacity issues with an SQL Server Reporting Services box hosted<br />
		on Microsoft's Azure Cloud Platform. The server had been setup around the time, Microsoft end-of-life'd<br />
		their platform-as-a-service report server offering, and forced everyone back onto standalone instances.<br />
		The server was a Basic A2 class VM (3.5GB Ram, 2 Cores). Originally, it only had to handle a small amount of report<br />
		creation load but in recent times, that load has gone up significantly. And due to the "peaky"<br />
		nature of the customer's usage, we would regularly see periods where the box could not keep up with<br />
		report generation requests.</p>
<p>In the past week, we've moved the customer to a new SQL Server 2014 Standard Edition install. Here are a few of the things we've<br />
		learned along the way with regards setting up SQL Server as a standalone instance on an Azure VM.</p>
<p><strong>This information is based on the service offerings and availabilities in the Azure North Europe region as of February 2016</strong></p>
<h2>Which Virtual Machine Class?</h2>
<p>First off, you should choose a <strong>DS</strong> scale virtual machine. At the time of writing, Microsoft offer 4 different VM classes<br />
	in the North Europe region: A, D, DS and D_V2. Only the DS class machines currently support Premium Locally<br />
	Redundant Storage (Premium LRS) which allows you to attach permanent SSD storage to your server.</p>
<p>Within the DS Set, DS1-DS4 have a slightly lower memory : core count ratio. The DS11-DS14 set have a higher<br />
	starting memory foot print for the same core count. We went with a DS3 server (4 core / 14GB) which we can downscale to<br />
	DS1 during out of hours periods.</p>
<p><img src="{{ site.baseurl }}/assets/01-which-vm.png" alt="Which Virtual Machine Class?" /></p>
<h2>Which Storage Account?</h2>
<p>During setup ensure that you’ve selected a Premium Locally Redundant Storage account which will<br />
	provide you access to additional attachable SSDs for your SQL Server. This can be found under<br />
	<i>Optional Configuration > Storage  > Create Storage Account > Pricing Tier</i></p>
<p><img src="{{ site.baseurl }}/assets/02-which-storage.png" alt="Which Storage Account?" /></p>
<h2>External Security</h2>
<p>Security will be somewhat dependent on your specific situation. In our case, this was a<br />
	standalone SQL Server with no failover cluster or domain management. The server was setup<br />
	with a long username and password (not the john.doe account in the screenshots).</p>
<p>We also lock down the management ports for Remote Desktop and Windows RM, as well as the added<br />
	HTTPS and SQL ports. To do this, add the public-to-private port mapping configurations under<br />
	Optional Configurations > Endpoints</p>
<p><img src="{{ site.baseurl }}/assets/03-endpoints.png" alt="Endpoint Configuration" /></p>
<p>Once you’ve finished setting up the configuration and azure has provisioned the server,<br />
	you’ll want to reenter the management blades and add ACL rules to lock down port access<br />
	to only the IP Ranges you want to access it. In our case, our development site, customer<br />
	site, and Azure hosted services. </p>
<p>You can add “permit” rules for specific IP addresses to access your server.<br />
	<a href="https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-acl/">Once a single<br />
	permit rule is added, all other IP Addresses/Ranges are blocked by Default. </a></p>
<p><img src="{{ site.baseurl }}/assets/04-endpoint-acl.png" alt="Endpoint ACLs" /></p>
<h2>Automated Backups</h2>
<p>SQL Azure VMs can now leverage an automated off-server database backup service<br />
	which will place your backups directly into Blob Storage. Select SQL Automated<br />
	Backup and enable it. You will be asked to specify where you would like to store your<br />
	backups and for how long. We chose to use a non-premium storage account<br />
	for this, and depending on the inherent value of your backups and whether you<br />
	intend to subsequently off-site them yourself, you might want to choose a storage<br />
	setup with zone or geo redundancy. You can also enable backup encryption by providing<br />
	a password here.</p>
<p><img src="{{ site.baseurl }}/assets/05-auto-backup.png" alt="Automated SQL Backup to Storage" /></p>
<h2>Disk Configuration</h2>
<p>Now that your server is up and running, you can log in via Remote Desktop. The first<br />
	thing you’ll want to do is patch your server. As of mid-February 2016, the base image<br />
	for SQL Server 2014 on Windows Server 2012 Standard R2 is missing quite a number of<br />
	patches. Approximately ~70 critical updates and another ~80 optional updates need to<br />
	be installed.</p>
<p>Once you’ve got your server patched, you can take a look at the disk setup. If you’ve<br />
	chosen a DS Class Server, you’ll notice that you have 2 Disks. A regular OS disk, and<br />
	an SSD Temp Disk. This temp disk is NOT to be used for real data, it is local only to<br />
	the VM while it’s running and will be deallocated and purged if you shut the server<br />
	down</p>
<p>You can however purchase additional SSD disks very easily. Head back out to the Azure<br />
	Management Portal, find your VM, go to settings and choose Disks. In the following<br />
	screenshot, we've chosen to add an additional 2 x 128GB disks (P10 class) disks to<br />
	the server. The <a href="https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-sql-server-performance-best-practices/"><br />
	SQL Server best practices</a> document recommends using the 1TB (P30 class) disks<br />
	which do give a significant I/O bump but they are also more expensive.</p>
<p>Ensure that you specify “Read Only” host caching for your Data Disk and No-Caching for<br />
	your Log disk to improve performance.</p>
<p><img src="{{ site.baseurl }}/assets/06-extra-disks.png" alt="Adding Extra Disks" /></p>
<p>Once your disks are attached you can access and map them inside your VM. We chose to<br />
	setup the disks using the newer Window Server 2012 Resilient File System (ReFS) rather<br />
	than NTFS. Previously there were potential issues with using ReFS in conjunction with<br />
	SQL Server, particularly in relation to sparse files and the use of DBCC CHECKDB however<br />
	these issues have been resolved in SQL Server 2014.</p>
<p><img src="{{ site.baseurl }}/assets/07-disk-config.png" alt="Disk Configuration" /></p>
<h2>Moving your Data Files</h2>
<p>SQL Server VM Images come pre-installed with SQL Server so we’ll need to do a little bit<br />
	of reconfiguration to make sure all our data and log files end up in the correct place. In the<br />
	following sections, disk letters & paths refer to the following.</p>
<ul>
<li>C: (OS Disk)</li>
<li>D:\SQLTEMP (Temp/Local SSD)</li>
<li>M:\DATA\ (Attached Perm SSD intended for Data)</li>
<li>L:\LOGS\ (Attached Perm SSD intended for Logs)</li>
</ul>
<p>First, we need to give permission to SQL Server to access these other disks. Assuming<br />
	you haven’t changed the default service accounts, then your SQL Server instance will<br />
	be running as the NT SERVICE\MSSQLSERVER account. You’ll need to give this account Full<br />
	Permissions on each of the locations you intend to store data and log files.</p>
<p><img src="{{ site.baseurl }}/assets/08-folder-permission.png" alt="Folder Permissions" /></p>
<p>Once the permissions are correct, we can specify those directories as new defaults<br />
	for our Data, Logs and Backups.</p>
<p><img src="{{ site.baseurl }}/assets/09-server-config.png" alt="Setting Default Paths for Data & Logs" /></p>
<p>Next We’ll move our master MDF and LDF files, by performing the following steps.</p>
<ol>
<li>Launch the SQL Server configuration Manager</li>
<li>Under SQL Server Services, select the main Server instance, and stop it</li>
<li>Right click the server instance, go to properties and review the startup parameters tab</li>
<li>Modify the –d and –e parameters to point to the paths where you intend to host your data and log files</li>
<li>Open Explorer and navigate to the default directory where the MDF files and LDF files are located (C:\Program Files\Microsoft SQL Server\MSSQL12.MSSQLSERVER\MSSQL\). Move the Master MDF and LDF to your new paths</li>
<li>Restart the Server</li>
</ol>
<p><img src="{{ site.baseurl }}/assets/10-master-move.png" alt="Moving the Master Database" /></p>
<p>When our server comes back online, we can move the remainder of the default databases.<br />
	Running the following series of SQL Commands will update the system to expect the MDFs<br />
	and LDFs and the new location on next start up.</p>
```sql
ALTER DATABASE [msdb] MODIFY FILE ( NAME = MSDBData , FILENAME = 'M:\DATA\MSDBData.mdf' )
ALTER DATABASE [msdb] MODIFY FILE ( NAME = MSDBLog , FILENAME = 'L:\LOGS\MSDBLog.ldf' )
ALTER DATABASE [model] MODIFY FILE ( NAME = modeldev , FILENAME = 'M:\DATA\model.mdf' )
ALTER DATABASE [model] MODIFY FILE ( NAME = modellog , FILENAME = 'L:\LOGS\modellog.ldf' )
ALTER DATABASE [tempdb] MODIFY FILE (NAME = tempdev, FILENAME = 'D:\SQLTEMP\tempdb.mdf');
ALTER DATABASE [tempdb] MODIFY FILE (NAME = templog, FILENAME = 'D:\SQLTEMP\templog.ldf');

--You can verify them with this command
SELECT name, physical_name AS CurrentLocation, state_desc FROM sys.master_files 
```
<p>Shut down the SQL Instance one more time. Phyiscally move your MDF and LDF files to<br />
	their new locations in Explorer, and finally restart the instance. If there are any<br />
	problems with the setup or the server fails to start, you can review the ERROR LOG in<br />
	C:\Program Files\Microsoft SQL Server\MSSQL12.MSSQLSERVER\MSSQL\Log\ERRORLOG</p>
<h2>Conclusions</h2>
<p><a href="https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-sql-server-performance-best-practices/"><br />
There are a number of other steps that you can then perform to tune your server.</a><br />
You should also setup SSL/TLS for any exposed endpoints to the outside world<br />
(e.g. if your going to run the server as an SSRS box). Hopefully you will have a you a far<br />
 more performant SQL Instance running in the Azure Cloud.</p>
<p><i>~Eoin Campbell</i></p>
