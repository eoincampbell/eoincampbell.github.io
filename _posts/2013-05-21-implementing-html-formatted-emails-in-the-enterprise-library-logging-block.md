---
layout: post
title: Implementing HTML Formatted Emails in the Enterprise Library Logging Block
date: 2013-05-21 12:35:59.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- C#
- Coding
tags:
- Custom Logging
- Email
- Enterprise Library
- Logging
- Tracelistener
meta:
  _edit_last: '1'
  _wpas_done_all: '1'
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p style="text-align: justify;"><a href="http://trycatch.me/blog/wp-content/uploads/2013/05/Unity.2.0.png"><img class="alignright size-full wp-image-847" alt="Enterprise Library" src="{{ site.baseurl }}/assets/Unity.2.0.png" width="100" height="100" /></a>The Microsoft Patterns &amp; Practices Enterprise Library contains a number of useful applications blocks for simplifying things like DataAcces, Logging &amp; Exception Handling in your .NET Applications. Recently we had a requirement to add HTML based formatting to the Email TraceListener in the Logging Application Block, something that's unfortunately missing from the base functionality. Thankfully, Enterprise Library is an open source code plex project so implementing a custom solution is a relatively trivial task. The email tracelistener functionality is contained in 3 main files: EmailTraceListener - The actual listener which you add to your configuration EmailTraceListenerData - The object representing the configuration settings EmailMessage - The wrapper object around a LogMessage which gets sent via email. Unfortunately because of the the way these classes are implemented in the EnterpriseLibrary Logging Block, they are not easily extended due to dependencies on Private Variables and Internal classes in the EnterpriseLibaray Logging Assembly so they need to be fully implemented in your own solution.</p>
<h2>Implementing a Solution</h2>
<p style="text-align: justify;">Step 1 was to take a copy of these three files and place them in my own Library Solution. I prefixed the name of each of them with Html; HtmlEmailTraceListener, HtmlEmailTraceListenerData and HtmlEmailMessage. Other code needed to be cleaned up including removing some dependencies on the internal ResourceDependency attributes used to decorate properties within the class &amp; tidying up the Xml Documentation Comments. The main change was then to enable the IsBodyHtml flag on the mail message itself. This was done in the CreateMailMessage method of the HtmlEmailMessage</p>
<pre class="brush:csharp; highlight: [20];">protected MailMessage CreateMailMessage()
{
	string header = GenerateSubjectPrefix(configurationData.SubjectLineStarter);
	string footer = GenerateSubjectSuffix(configurationData.SubjectLineEnder);

	string sendToSmtpSubject = header + logEntry.Severity.ToString() + footer;

	MailMessage message = new MailMessage();
	string[] toAddresses = configurationData.ToAddress.Split(';');
	foreach (string toAddress in toAddresses)
	{
		message.To.Add(new MailAddress(toAddress));
	}

	message.From = new MailAddress(configurationData.FromAddress);

	message.Body = (formatter != null) ? formatter.Format(logEntry) : logEntry.Message;
	message.Subject = sendToSmtpSubject;
	message.BodyEncoding = Encoding.UTF8;
	message.IsBodyHtml = true;

	return message;
}</pre>
<h2>Using your new solution</h2>
<p style="text-align: justify;">Once implemented it's simply a matter of reconfiguring your app/web.config logging sections to use the new types you've created instead of the original Enterprise Library types. You need to change the type and listenerDataType properties of your Email Listener in the &amp;gl;listeners@gt; section of your config.</p>
<pre class="brush: xml; highlight: [16, 17];">&lt;listeners&gt;
      &lt;!-- Please update the following Settings: toAddress, subjectLineStarter, subjectLineEnder--&gt;
      &lt;add name="EmailLog"
           toAddress="toAddress@example.com"
           subjectLineStarter="Test Console - "
           subjectLineEnder=" Alert"
           filter="Verbose"
           fromAddress="fromAddress@example.com"
           formatter="EmailFormatter"
           smtpServer="smtp.gmail.com"
           smtpPort="587"
           authenticationMode="UserNameAndPassword"
           useSSL="true"
           userName="fromAddress@example.com"
           password="Password"
           type="YourLibrary.YourNamespace.HtmlEmailTraceListener, YourLibrary, Version=1.0.0.0, Culture=neutral, PublicKeyToken=0000000000000000"
           listenerDataType="YourLibrary.YourNamespace.HtmlEmailTraceListenerData,  YourLibrary, Version=1.0.0.0, Culture=neutral, PublicKeyToken=0000000000000000"
           traceOutputOptions="Callstack" /&gt;
    &lt;/listeners&gt;</pre>
<p style="text-align: justify;">You'll also need to ensure that you've escaped your Html formatted textFormatter template in the formatters section of your code. i.e. replacing &lt;html&gt; with &amp;lt;html&amp;gt;</p>
<pre class="brush: xml; highlight: [4];">&lt;formatters&gt;
      &lt;add name="EmailFormatter" 
              type="Microsoft.Practices.EnterpriseLibrary.Logging.Formatters.TextFormatter, Microsoft.Practices.EnterpriseLibrary.Logging, Version=5.0.414.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" 
              template="
                  &amp;lt;html&amp;gt;
                  &amp;lt;body&amp;gt;
                  &amp;lt;table border=&amp;quot;1&amp;quot; style=&amp;quot;border: solid 1px #000000; border-collapse:collapse;&amp;quot;&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;&amp;lt;b&amp;gt;Message&amp;lt;/b&amp;gt;&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;&amp;lt;b&amp;gt;{message}&amp;lt;/b&amp;gt;&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;Local TimeStamp&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{timestamp(local)}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;Timestamp&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{timestamp}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;Title&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{title}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;Severity&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{severity}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;Category&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{category}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;Priority&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{priority}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;EventId&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{eventid}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;Local Machine&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{localMachine}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;AppDomain&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{appDomain}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;LocalDomain&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{localAppDomain}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;Local Process Name&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{localProcessName}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;Local Process&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{localProcessId}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;Win32ThreadId&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{win32ThreadId}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;ThreadName&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{threadName}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;Extended Properties&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;
                  &amp;lt;table border=&amp;quot;1&amp;quot; style=&amp;quot;border: solid 1px #000000; border-collapse:collapse;&amp;quot;&amp;gt;
                  {dictionary(&amp;lt;tr&amp;gt;&amp;lt;td&amp;gt;{key}&amp;lt;/td&amp;gt;&amp;lt;td&amp;gt;{value}&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;)}
                  &amp;lt;/table&amp;gt;&amp;lt;/td&amp;gt;&amp;lt;/tr&amp;gt;
                  &amp;lt;/table&amp;gt;
                  &amp;lt;/body&amp;gt;
                  &amp;lt;/html&amp;gt;
" /&gt;

    &lt;/formatters&gt;</pre>
<p>All done. Now you can happily send email log messages in HTML format via your Application Logging calls.</p>
<p><em>~EoinC</em></p>
