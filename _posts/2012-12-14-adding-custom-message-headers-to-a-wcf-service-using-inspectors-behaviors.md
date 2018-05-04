---
layout: post
title: Adding Custom Message Headers to a WCF Service using Inspectors & Behaviors
date: 2012-12-14 13:47:21.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- Coding
tags:
- Custom Header
- fiddler
- Header
- Message
- Message Header
- MessageInspector
- Service Behavior
- soap header
- wcf
meta:
  _edit_last: '1'
  _syntaxhighlighter_encoded: '1'
  _wpas_done_all: '1'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1524518710;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:580;}i:1;a:1:{s:2:"id";i:846;}i:2;a:1:{s:2:"id";i:46;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><img class=" wp-image-778 " title="WCF Header Man" alt="WCF Header Man" src="{{ site.baseurl }}/assets/wcfheader.png" width="105" height="223" />He has a WCF Header... Get it </p>
<p style="text-align: justify;">Often, you'll need to pass some piece of information on some or all of your  WCF Service operations. For my team, we had recently exposed some functionality in an old WCF Endpoint via a Web Front End and wanted to log some auditing information on each and every Service Call. Obviously modifying every single method signature to accept the new parameters would be a pretty significant breaking change to all the consumers so instead we looked at passing this information as a Custom WCF Message Header. WCF exposes a number of interfaces which you can leverage to inspect &amp; modify messages  on the fly and to add customer behaviors to service endpoints. In the following demo, we'll go through the process of building a Custom Header for our WCF Service &amp; subsequently passing that information from the Consumer Client back to the service. In our contrived example I'll be attempting to pass 3 pieces of information to the WCF Service as part off every message call.</p>
<ul>
<li>The username of the currently logged in web front-end user</li>
<li>Since our website is deployed across multiple nodes, the id of the web node</li>
<li>The "Special Session Guid" of the current users Session</li>
</ul>
<div style="text-align: justify;">[important]The completed solution can be found on GitHub at <a href="https://github.com/eoincampbell/wcf-custom-headers-demo" target="_blank">https://github.com/eoincampbell/wcf-custom-headers-demo</a> [/important]</div>
<div style="text-align: justify;"></div>
<div style="text-align: justify;">This scenario could apply to a number of other real world situations. Perhaps the service is secured and called using a single WS Security Account, but we need to log the user who's session, the service call originated from on every service call. Or perhaps you're exposing your service to the public and as well as providing a username &amp; password to authenticate, the caller also needs to provide some sort of "Subscription Account Number" in addition to their Credentials. Any of these scenarios are candidates for adding a custom header to a WCF Service.</div>
<div></div>
<h1>The Quick &amp; Dirty Solution</h1>
<div style="text-align: justify;">Of course I could just edit method signatures of each service call to accept this additional information as additional parameters but this causes a number of other problems. This might be feasible for a small in-house service, or dummy application but it causes a number of issues in reality.</div>
<div>
<ul>
<li style="text-align: justify;">I need to add additional parameters to every service call which isn't a particularly elegant solution</li>
<li style="text-align: justify;">If I need to add more information in the future, I must edit every service call signature which at worst is a breaking change for every client and at best, a significant amount of work &amp; re-factoring.</li>
<li style="text-align: justify;">Depending on the scenario, I'm potentially intermingling Business Logic with Authentication/Authorization or some other sort of Service Wide Validation logic which is going to increase the complexity of any future re-factoring.</li>
</ul>
</div>
<h1>Operation Context &amp; Custom Message Headers</h1>
<p style="text-align: justify;">A better solution would be to "tag" each service call with some sort of header information. In this way, we could piggy-back our additional data along without interfering with the individual method signatures of each service call. Thankfully WCF includes built-in support for MessageHeaders. The services OperationContext includes Incoming &amp; Outgoing Message Header collections.</p>

```csharp
//Service Contract
[ServiceContract]
public interface ISimpleCustomHeaderService
{
    [OperationContract]
    void DoWork();
}
//Client Code
using (var client = new SimpleCustomHeaderServiceClient())
using (var scope = new OperationContextScope(client.InnerChannel))
{
    var webUser = new MessageHeader("joe.bloggs");
    var webUserHeader = webUser.GetUntypedHeader("web-user", "ns");
    OperationContext.Current.OutgoingMessageHeaders.Add(webUserHeader);
    client.DoWork();
}
```

<p style="text-align: justify;">For now I've created a very simple Service Contract which has a single void method on it called <strong>DoWork()</strong>. Adding a custom header to this service call is relatively trivial. First we instantiate a new instance of our WCF Client Proxy. We also nee to create an <a title="OperationContextScope Class (System.ServiceModel)" href="http://msdn.microsoft.com/en-us/library/system.servicemodel.operationcontextscope.aspx" target="_blank"><strong>OperationContextScope </strong></a>using the WCF client channel. Since the <strong>OperationContext </strong>is accessed, via the static Current property, instantiating this scoping object, stores the current context &amp; the <strong>OperationContext</strong> of the current Clients <strong>IContextChannel</strong> becomes that returned by the Current Property. This allows us to modify the <strong>OutgoingMessageHeaders</strong> collection of the clients channel. Once disposed the state of the original Current <strong>OperationContext</strong> is restored. MessageHeaders are passed as untyped data as they travel on the wire. In the example above I've created a strongly typed .NET object. That is then converted to an untyped header; keyed by name &amp; namespace for transmission in the <strong>OutgoingMessageHeaders</strong> Collection. If we observe the data that travels across the wire using fiddler, we can see our Custom Header Data has been appended to the soap header section of the message.</p>
<p><img class=" wp-image-762 " title="Fiddler WCF Headers" alt="Fiddler WCF Headers" src="{{ site.baseurl }}/assets/fiddler-wcf-header.png" width="457" height="405" />Fiddler WCF Headers</p>
<p style="text-align: justify;">Finally, these Header values can be retrieved from the <strong>IncomingMessageHeader</strong> Collection as part of the service call processing. Since we're already in the scope the Current OperationContext, we can just directly access that context's header collection to read our headers. I've added a simple generic helper method to test to see if the Header can first be found and if so, will be returned.</p>

```csharp
public class SimpleCustomHeaderService : ISimpleCustomHeaderService
{
    public string DoWork()
    {
        //Do Work
        //...
        //Capture Headers
        var userName = GetHeader("web-user", "ns");
        var webNodeId = GetHeader("web-node-id", "ns");
        var webSessionId = GetHeader("web-session-id", "ns");
        Debug.WriteLine("User: {0} / Node: {1} / Session: {2}", userName, webNodeId, webSessionId);
        var s = string.Format("HeaderInfo: {0}, {1}, {2}",
            userName,
            webNodeId,
            webSessionId);</p>
        return s;
    }
        private static T GetHeader(string name, string ns)
    {
        return OperationContext.Current.IncomingMessageHeaders.FindHeader(name, ns) > -1
            ? OperationContext.Current.IncomingMessageHeaders.GetHeader(name, ns)
            : default(T);
    }
}
```

<h1>Leveraging Client &amp; Dispatch Message Inspectors</h1>
<p style="text-align: justify;">The above solution comes with some pros and cons. On the plus side headers can be added in an adhoc manner with little friction to existing code. On the downside, it's not really ideal from a code maintenance/organization point of view. Header Data is relatively unstructured and disparate. We also end up with a lot of touch-points. Adding headers requires creating an <strong>OperationContextScope</strong> in close proximity to every service call... similarly, accessing the resultant header values must be done in the Service Methods... Imagine our WCF Service had 100 service methods, and all we wanted to do was send a single additional header to be logged on the server. That results in 100's of lines of additional code.</p>
<p style="text-align: justify;">A better solution would be to use the built in message inspector interfaces in WCF. Message Inspectors provide you with a way to plug directly into the WCF Communications Pipeline on both the client or server side of the communication Channel. The <strong>IDispatcherMessageInspector</strong> allows us to affect messages either just after the request has arrives on the server (<strong>AfterReceiveRequest</strong>) or just before the response leaves the server (<strong>BeforeSendReply</strong>) The <strong>IClientMessageInspector</strong> allows us to affect messages either just before the request leaves the client  (<strong>BeforeSendRequest</strong>) or just after the response is received by the client (<strong>AfterReceiveReply</strong>)</p>

```csharp
public class CustomInspectorBehavior : IDispatchMessageInspector, IClientMessageInspector
{
    #region IDispatchMessageInspector</p>
    public object AfterReceiveRequest(ref Message request, IClientChannel channel, InstanceContext instanceContext)
    { ... }
    #endregion
    #region IClientMessageInspector
    public object BeforeSendRequest(ref Message request, IClientChannel channel)
    { ... }
    #endregion
}
```

<p style="text-align: justify;">Injecting ourselves into the message pipeline like this serves a number of advantages, We now have the opportunity to add our messages to the outbound client request in a single place. We also have a single touch point for capturing the request on the server side. If this was a licence-code validation check, this would save us from peppering every single service call with the validation check code.</p>
<p style="text-align: justify;">In the following sections we'll look at creating a custom data header object, creating a message inspector implementation to manage injecting and extracting this data from our WCF Service, creating Client &amp; Service Behaviors to attach our Message Inspectors and creating a behavior extension to allow these behaviors to be applied to our service through configuration.</p>
<h1 style="text-align: justify;">Solution Organisation</h1>
<p>Since both our Web Application (which will host the Web Services) and the Console Application will have some common dependencies, I've split out the majority of the code in these next sections and stored them in Common Class Library folder which both Applications can then reference.</p>
<p><img class=" wp-image-767 " alt="Solution Organisation" src="{{ site.baseurl }}/assets/sol.png" width="212" height="224" />Solution Organisation</p>
<h1>Custom Header Data Contract</h1>
<p style="text-align: justify;">The first thing I'll create is a simple data contract object to represent our custom header information. This POCO Data Contract provides a simple way for us to encapsulate our header information into a single payload which will be transmitted with our Service Calls.</p>

```csharp
    [DataContract]
    public class CustomHeader
    {
        [DataMember]
        public string WebUserId { get; set; }
        [DataMember]
        public int WebNodeId { get; set; }
        [DataMember]
        public Guid WebSessionId { get; set; }
    }
``` 

<h1>Message Inspectors</h1>
<p style="text-align: justify;">Next I create our Message Inspectors. The two interfaces that are required are the <strong>System.ServiceModel.Dispatcher.IDispatchMessageInspector</strong> (which hooks into our pipeline on the service side) and the  <strong>System.ServiceModel.Dispatcher.IClientMessageInspector</strong> (which hooks into our pipeline on the consumer side). Within these two interfaces the two methods I'm most interested in are the <strong>IClientMessageInspector.BeforeSendRequest</strong> which allows me to modify the outgoing header collection on the client and the <strong>IDispatchMessageInspector.AfterReceiveRequest </strong>which allows me to retrieve the data on the service side.</p>

```csharp
    #region IDispatchMessageInspector
    public object AfterReceiveRequest(ref Message request, IClientChannel channel, InstanceContext instanceContext)
    {
        //Retrieve Inbound Object from Request
        var header = request.Headers.GetHeader("custom-header", "s");
        if (header != null)
        {
            OperationContext.Current.IncomingMessageProperties.Add("CustomHeader", header);
        }
        return null;
    }
    #endregion</p>
    #region IClientMessageInspector
    public object BeforeSendRequest(ref Message request, IClientChannel channel)
    {
        //Instantiate new HeaderObject with values from ClientContext;
        var dataToSend = new CustomHeader
            {
                WebNodeId = ClientCustomHeaderContext.HeaderInformation.WebNodeId,
                WebSessionId = ClientCustomHeaderContext.HeaderInformation.WebSessionId,
                WebUserId = ClientCustomHeaderContext.HeaderInformation.WebUserId
            };</p>
        var typedHeader = new MessageHeader(dataToSend);
        var untypedHeader = typedHeader.GetUntypedHeader(&quot;custom-header&quot;, &quot;s&quot;);</p>
        request.Headers.Add(untypedHeader);
        return null;
    }
    #endregion
```

<p style="text-align: justify;">I'll also need to create a simple static Client Context class which will provide the conduit for the Consumer Application to set header values to be picked up inside the message inspector methods.</p>

```csharp
    public static class ClientCustomHeaderContext
    {
        public static CustomHeader HeaderInformation;
        static ClientCustomHeaderContext()
        {
            HeaderInformation = new CustomHeader();
        }
    }
```

<h1 style="text-align: justify;">WCF Custom Behaviors</h1>
<p style="text-align: justify;">WCF Service Behaviors define how the endpoint (the actual service instance) interacts with its clients. Attributes like security, concurrency, caching, logging, and attached message inspectors - those are all part of the behavior. We're going to implement a new custom behavior for both the Service side and the Client side of this interaction. Since these are still just interface implementations, there's no need to create a new class to implement them. We can add this functionality to the same class which contains our Message Inspector functionality. Of course if you wanted to be a purist about it, there's nothing to stop you implementing the two message inspectors and two service behaviors in four completely separate classes.</p>
<p style="text-align: justify;">The two behavior contracts I'm interested in here are the <strong>System.ServiceModel.Description.IEndpointBehavior</strong>, which is responsible for the client side behavior and the <strong>System.ServiceModel.Description.IServiceBehavior</strong> which is responsible for the service side behavior. Implementing these interfaces allows me to add an instance of the Message Inspectors to the service.</p>

```csharp
#region IEndpointBehavior

public void ApplyDispatchBehavior(ServiceEndpoint endpoint, EndpointDispatcher endpointDispatcher)
{
    var channelDispatcher = endpointDispatcher.ChannelDispatcher;
    if (channelDispatcher == null) return;
    foreach (var ed in channelDispatcher.Endpoints)
    {
        var inspector = new CustomInspectorBehavior();
        ed.DispatchRuntime.MessageInspectors.Add(inspector);
    }
}

public void ApplyClientBehavior(ServiceEndpoint endpoint, ClientRuntime clientRuntime)
{
    var inspector = new CustomInspectorBehavior();
    clientRuntime.MessageInspectors.Add(inspector);
}

#endregion

#region IServiceBehaviour

public void ApplyDispatchBehavior(ServiceDescription serviceDescription, ServiceHostBase serviceHostBase)
{
    foreach (ChannelDispatcher cDispatcher in serviceHostBase.ChannelDispatchers)
    {
        foreach (var eDispatcher in cDispatcher.Endpoints)
        {
            eDispatcher.DispatchRuntime.MessageInspectors.Add(new CustomInspectorBehavior());
        }
    }
}

#endregion
```

<h1 style="text-align: justify;">Adding the Custom Behavior to a WCF Service</h1>
<p style="text-align: justify;">Behaviors can be applied to services using a special Service Behavior attribute which decorates the ServiceContract. The last step is to extend the Attribute class in our CustomHeaderInspectorBehavior class and then to decorate each of services with that attribute.</p>

```csharp
[AttributeUsage(AttributeTargets.Class)]
public class CustomInspectorBehavior : Attribute, ... { ... }
[CustomInspectorBehavior]
public class ComplexCustomHeaderService : IComplexCustomHeaderService { ... }
```

<h1 style="text-align: justify;">Configuring a WCF Client to use a specific behavior</h1>
<p style="text-align: justify;">On the client side, I need to do a tiny bit more work. I can manually configure the Behavior on the WcfClientProxy every time I instantiate it but this is extra bloat and eventually I'll forget to set it somewhere and lose my behavior functionality.</p>

```csharp
using(var client = new ComplexCustomHeaderServiceClient()) {
    client.ChannelFactory.Endpoint.Behaviors.Add(new CustomHeaderInspectorBehavior());
}
```

<p style="text-align: justify;">Instead I'd prefer to be able to set this once in configuration and never have to worry about it again. I can achieve this by using a BehaviorExtension Element as follows and adding it to my application configuraiton file.</p>

```csharp
public class CustomInspectorBehaviorExtension : BehaviorExtensionElement
{
    protected override object CreateBehavior()<
    {
        return new CustomInspectorBehavior();
    }
    public override Type BehaviorType
    {
        get { return typeof (CustomInspectorBehavior);}
    }
}
```

<p style="text-align: justify;">And below is the equivalent configuration file.</p>

```xml
    <system.serviceModel>
      <behaviors>
        <endpointBehaviors>
          <behavior name="CustomInspectorBehavior">
            <CustomInspectorBehavior />
          </behavior>
        </endpointBehaviors>
      </behaviors>
      <extensions>
        <behaviorExtensions>
          <add name="CustomInspectorBehavior"
               type="WCFCustomHeaderDemo.Lib.Extensions.CustomInspectorBehaviorExtension,WCFCustomHeaderDemo.Lib" />
        </behaviorExtensions>
      </extensions>
        <bindings>
            <basicHttpBinding>
                <binding name="BasicHttpBinding_ISimpleCustomHeaderService" />
                <binding name="BasicHttpBinding_IComplexCustomHeaderService" />
            </basicHttpBinding>
        </bindings>
        <client>
            <endpoint address="http://localhost/TestService/ComplexCustomHeaderService.svc"
                behaviorConfiguration="CustomInspectorBehavior"
				binding="basicHttpBinding"
                bindingConfiguration="BasicHttpBinding_IComplexCustomHeaderService"
                contract="ComplexCustomHeaderService.IComplexCustomHeaderService"
                name="BasicHttpBinding_IComplexCustomHeaderService" />
        </client>
    </system.serviceModel>
```

<h1 style="text-align: justify;">Calling our Client</h1>
<p>Finally, we can call our client and test to see if our Server side application can see the headers being submitted and echo them back.</p>

```csharp
using(var client = new ComplexCustomHeaderServiceClient())
{
    ClientCustomHeaderContext.HeaderInformation.WebNodeId = 465;
    ClientCustomHeaderContext.HeaderInformation.WebSessionId = Guid.NewGuid();
    ClientCustomHeaderContext.HeaderInformation.WebUserId = "joe.bloggs"
    System.Console.WriteLine(client.DoWork());
}

```

<p><img class=" wp-image-774 " alt="Wcf Header Demo Result" src="{{ site.baseurl }}/assets/result.png" width="510" height="147" />WCF Header Demo Result</p>
<p>Excellent.</p>
<p><em>~Eoin C</em></p>
