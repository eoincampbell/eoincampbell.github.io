---
layout: post
title: Xml & Json Serialization of Object Graphs with Cyclic References in .NET
date: 2012-08-30 08:14:01.000000000 +01:00
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
- ".net"
- ".net4.5"
- circular
- cyclic
- datacontract
- datacontractserializer
- json
- reference
- serialization
- wcf
- XML
meta:
  _edit_last: '1'
  _thumbnail_id: '598'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1525001726;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:747;}i:1;a:1:{s:2:"id";i:835;}i:2;a:1:{s:2:"id";i:369;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><img class=" wp-image-598  " title="Cyclic Reference" src="{{ site.baseurl }}/assets/cref.jpg" alt="Cyclic Reference" width="192" height="192" /> Cyclic Reference</p>
<p style="text-align: justify;">As a WCF Developer you've no doubt run into some issues with object serialization at some point or another during the course of developing your applications. Either during the course of pushing your entities through a WCF Service or by attempting to serialize those object for transit through other media. Recently I ran into some issues around serializing WCF Entities with Circular References (or cyclic relationships) between parent &amp; child Objects.</p>
<h1 style="text-align: justify;">Background</h1>
<p style="text-align: justify;">A large percentage of our company's software work relies on moving a variety of object graphs around through WCF. These object graphs, often times contain self-references or circular references. The data we deal with is primarily related to the airline industry; specifically crew &amp; flight schedules. It contains complicated tree structures of data with information on the flights crew-member's fly, the duties (work-shifts) those flights belong to &amp; the pairings, those duties are contained within. (A pairing is airline industry parlance for a collection of Duties organised into a longer 4 or 5 scheduled body of work for a crew member. Confusingly, it is not necessarily a <em>Pair</em> of anything).</p>
<p style="text-align: justify;">In order to allow us to more easily navigate these object graphs, we populate a number of helper properties on the entities to allow us to traverse up and down the graph with ease. The following is an extremely simplified example of what gets populated in our Crew Schedule "Roster" Object Graph and shows the relations ships between the three previously mentioned domain entities.</p>
<p style="text-align: justify;">Of course, this makes for some interesting cyclic situations when both the parent objects &amp; contents of the child collections are cross-wired to one another and we want to serialize those objects for use through a WCF Service.</p>
<p style="text-align: justify;"><!--more--></p>

```csharp
public class Pairing
{
    public IList Duties { get; set; }
}

public class Duty
{
    public Pairing ParentPairing { get; set; }
    public IListFlights { get; set; }
}

public class Flight
{
    public Duty ParentDuty { get; set; }
}

...

Pairing p = Pairing.LoadFromSource();
p.Duties[0].ParentPairing.Duties[0].ParentPairing.ItsTurtlesAllTheWayDown();
```

<h1>Xml Serialization with DataContractSerializer</h1>
<p>&nbsp;</p>
<p style="text-align: justify;">These kind of circular relationships, play havoc when it comes to attempting to serialize an object graph. Recently, we have been looking at different ways to pass message objects to other systems. These include other .net applications which are interconnected by WCF Services &amp; various message queueing technologies (RabbitMQ, ActiveMQ, MSMQ etc...) as well as talking to non .NET applications where we'd like to provide the data in as technology agnostic a format as possible (i.e. Json).</p>
<p style="text-align: justify;">Our first attempts at XML Serialization were to use the basic <strong><code>XMLSerializer</code></strong> and flag certain parent or child properties with the <strong><code>[XmlIgnore]</code></strong> Attribute. However this causes a issues for the consumer of the data as they would have to rematerialize the object and then re-walk the entire object graph, re-wiring the correct relationships.</p>
<p style="text-align: justify;">A better solution was to use the  System.Runtime.Serialization.DataContractSerializer. Simply decorate your classes with the <strong><code>[DataContract]</code></strong> attribute, and the members you want serialized with the <strong><code>[DataMember]</code></strong> attribute. Lets create a simple Family/Parent/Child object graph for demonstration.</p>
<h2 style="text-align: justify;">Entity Model</h2>

```csharp
[DataContract]
public class Family
{
    [DataMember]    public IList<Parent&gt; Parents;
    [DataMember]    public IList<Child&gt; Children;
}

[DataContract]
public class Parent
{
    [DataMember]    public string Name { get; set; }
    [DataMember]    public IList<Child&gt; Children { get; set; }
}

[DataContract]
public class Child
{
    [DataMember]    public string Name { get; set; }
    [DataMember]    public Parent Father { get; set; }
    [DataMember]    public Parent Mother { get; set; }
}
```

<h2 style="text-align: justify;">Sample Usage</h2>

```csharp
var dad = new Parent { Name = "John" };
    var mum = new Parent { Name = "Mary" };

    var kid1 = new Child { Name = "Ann", Mother = mum, Father = dad };
    var kid2 = new Child { Name = "Barry", Mother = mum, Father = dad };
    var kid3 = new Child { Name = "Charlie", Mother = mum, Father = dad };

    var listOfKids = new List {kid1, kid2, kid3};
    dad.Children = listOfKids;
    mum.Children = listOfKids;

    var family = new Family { Parents = new List {mum, dad}, Children = listOfKids };
```

<h2 style="text-align: justify;">Object Graph Contains Cycles</h2>

```csharp
var serializedData = string.Empty;

//Serialize
using (var ms = new MemoryStream())
{
    var serializer = new DataContractSerializer(typeof(Family));
    serializer.WriteObject(ms, family);
    serializedData = Encoding.UTF8.GetString(ms.ToArray());
    Console.WriteLine(serializedData);
}

//Deserialize
using (var ms = new MemoryStream(Encoding.UTF8.GetBytes(serializedData)))
{
    var serializer = new DataContractSerializer(typeof(Family));
    var f = (serializer.ReadObject(ms) as Family) ?? new Family();
    Console.WriteLine(f.ToString());
}
```

<p style="text-align: justify;">The first issue we run into is an Unhandled <strong><code>System.Runtime.Serialization.SerializationException. Object graph for type 'ConsoleApplication1.Child' contains cycles and cannot be serialized if reference tracking is disabled</code></strong>. This occurs because we have a circular relationship between each child object &amp; each parent object. The solution to this is to mark each of the objects data contracts in our as Referential Types.</p>

```csharp
[DataContract(IsReference = true)]
public class Child
{
    ...
}
```

<p style="text-align: justify;">The output format for this can be a little hard to understand as objects are serialized to XML the first time they are encountered during the serializer's walk of the object graph and then tagged with a Ref Id. Each subsequent encounter records only the reference id and does not enumerate the entire sub-object graph.</p>

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Family z:Id="i1" xmlns="http://schemas.datacontract.org/2004/07/ConsoleApplication1" xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns:z="http://schemas.microsoft.com/2003/10/Serialization/">
  <Children>
    <Child z:Id="i2">
      <Father z:Id="i3">
        <Children>
          <Child z:Ref="i2" />
          <Child z:Id="i4">
            <Father z:Ref="i3" />
            <Mother z:Id="i5">
              <Children>
                <Child z:Ref="i2" />
                <Child z:Ref="i4" />
                <Child z:Id="i6">
                  <Father z:Ref="i3" />
                  <Mother z:Ref="i5" />
                  <Name>Charlie</Name>
                </Child>
              </Children>
              <Name>Mary</Name>
            </Mother>
            <Name>Barry</Name>
          </Child>
          <Child z:Ref="i6" />
        </Children>
        <Name>John</Name>
      </Father>
      <Mother z:Ref="i5" />
      <Name>Ann</Name>
    </Child>
    <Child z:Ref="i4" />
    <Child z:Ref="i6" />
  </Children>
  <Parents>
    <Parent z:Ref="i5" />
    <Parent z:Ref="i3" />
  </Parents>
</Family>
```

<h1 style="text-align: justify;">Json Serialization - NewtonSoft Json.NET to the rescue</h1>
<p style="text-align: justify;">Json Serialization can be achieved in a number of ways in .NET 4.5 using either the AJAX Javascript Serializer or the Serialization DataContractJsonSerializer. Unfortunately we get stuck between a rock and a hard place using the DataContractJSonSerializer. If we attempt to use the Serializer in the same was as above, we'll be met with the following exception.</p>
<p style="text-align: justify;"><strong><code>System.Runtime.Serialization.SerializationException was unhandled. The type 'ConsoleApplication1.Family' cannot be serialized to Json because its IsReference setting is 'True'. The JSON format does not support references because there is no standardized format for representing references. To enable serialization, disable the IsReference setting on the type or an appropriate parent class of the type.</code></strong></p>
<p style="text-align: justify;">Of course if we disable the IsReference property of our contract attrribute we end up back in our original conundrum facing a Cyclic reference.</p>
<p style="text-align: justify;">In the end, we opted to use <a title="Json.NET" href="http://james.newtonking.com/projects/json-net.aspx" target="_blank">Json.NET by James Newtown King</a>. Json.NET does have support for referential types containing cyclic relationships. You can use Json.Net in your applications by adding the package via Nuget.</p>
<p>&nbsp;</p>
<div class="nuget-badge" style="padding-left: 90px; padding-right: 120px;"><code>PM&gt; Install-Package NewtonSoft.Json </code></div>
<p>&nbsp;</p>
<p style="text-align: justify;">Once referenced; serializing your objects to Json can be done in the following way.</p>

```csharp
var jsonSerializer = new JsonSerializer
{
    NullValueHandling = NullValueHandling.Ignore,
    MissingMemberHandling = MissingMemberHandling.Ignore,
    ReferenceLoopHandling = ReferenceLoopHandling.Serialize
};

var sb = new StringBuilder();
using (var sw = new StringWriter(sb))
    using (var jtw = new JsonTextWriter(sw))
        jsonSerializer.Serialize(jtw, family);

var result = sb.ToString();
Console.WriteLine(result);
```

<p>&nbsp;</p>
<p style="text-align: justify;">Works just great.<br />
<em>~Eoin Campbell</em></p>
