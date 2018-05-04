---
layout: post
title: Building Lambda Expressions at Runtime
date: 2010-11-30 15:26:42.000000000 +00:00
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
- clause
- dynamic
- expression
- lambda
- linq
- linq2sql
- parameterexpression
- runtime
- where
meta:
  _wp_old_slug: ''
  _edit_last: '1'
  _thumbnail_id: '387'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1525348247;s:7:"payload";a:3:{i:0;a:1:{s:2:"id";i:876;}i:1;a:1:{s:2:"id";i:867;}i:2;a:1:{s:2:"id";i:580;}}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
excerpt: Necessity is the mother of all... reasons to learn something new. So when
  some project requirements came down to put together a Search UI for an object graph
  of ~200 different properties in one wide table, we got an opportunity to play with
  some dynamic LINQ. We needed to come up with a quick way to allow them to choose
  between which properties they wanted to search by, the operators applicable to that
  properties type.
---
<p>[caption id="attachment_387" align="alignright" width="300" caption="Dynamic"]<img src="{{ site.baseurl }}/assets/dynamic-quaternity-cosmic-redemption-web-300x225.jpg" alt="Dynamic" title="Dynamic" width="300" height="225" class="size-medium wp-image-387" />[/caption]
<p style="text-align: justify;">Necessity is the mother of all... reasons to learn something new. So when some project requirements came down to put together a Search UI for an object graph of ~200 different properties in one wide table, we got an opportunity to play with some dynamic LINQ. We needed to come up with a quick way to allow a user to search across all the properties without making the UI unwieldy. What we provided them with was a simple UI allowing the user to apply 0:N conjunctive search filters. For each filter they choose an object property to filter by, the filtering operator (equal, less than, etc...) and the value they were searching for.</p>
<p style="text-align: justify;">By the way, if there's a nicer way to do this, I'd love to know about it.</p>
<p><!--more--></p>
<p style="text-align: justify;">To get our list of properties they could choose from was relatively simple... Since we already had a Linq2SQL mapping of our Device table, we could just call the GetProperties method on that entity and stick the resultant collection in a ComboBox.</p>

```csharp
Dictionary<string, string> returned = typeof(Device).GetProperties();
```
<p style="text-align: justify;">But how could we use these string representation of object properties in a dynamic fashion. First off, lets have a look at what LINQ is actually doing. Lets take a very simple example.</p>

```csharp
var results = dataContext.Devices.Where(device => device.DeviceManufacturerId == 123);
```

<p style="text-align: justify;">It doesn't really lend itself nicely to dynamically altering the where clause at runtime. Lucky for us, the query above is really just syntactic sugar and the System.Linq namespace provides us with everything we need to concoct our own lambdas at runtime.</p>
<h4>Query Preparation</h4>
<p style="text-align: justify;">We want to return a queryable collection of Device Objects. Since the lambda execution is delayed until we attempt to interact with the results, we can safely build up our query in successive steps.</p>

```csharp
IQueryable<Device> devices dataContext.Devices;
```
<p style="text-align: justify;">Next we created a <code>ParameterExpression</code> to reference our properties in our Dynamic Lambda.</p>

```csharp
ParameterExpression parameterExpression = Expression.Parameter(
   typeof(Device), 
   "device"
);
```

<p style="text-align: justify;">This is equivalent to the <em><strong>"device =&gt;"</strong></em> portion of the query.</p>
<p style="text-align: justify;">We also added our good old "WHERE 1 = 1" hack. Since our UI allowed a user to search without applying any filter, we didn't want to have to worry about testing for a pre-existing clause before adding the next conjunction predicate.</p>

```csharp
Expression whereClauseExpression = Expression.Equal(
   Expression.Constant(1), 
   Expression.Constant(1)
);
```

<p style="text-align: justify;">Right now our query looks like this and any user generated filters can be easily AND'd on</p>

```csharp
var results = dataContext.Devices.Where(
   device => 1 == 1
);
```

<h4>The Operators</h4>
<p style="text-align: justify;">Since the operations that the user was going to use in our UI were well defined, we opted to simplify this part of the system.<br />
numbers: ==, &lt;, &gt;, &lt;=, &gt;=, !=<br />
strings: ==, contains<br />
bools: ==, !=</p>
<p style="text-align: justify;">We created a delegate to represent an Expression Operator</p>

```csharp
delegate BinaryExpression OperatorDelegate(
   Expression left, 
   Expression right
);
```

<p style="text-align: justify;">And then mapped the users selections to the actual operators.</p>

```csharp
Dictionary ops = new Dictionary();
ops.Add("==", Expression.Equal);
//etc...
```

<h4>The dynamagic bit</h4>
<p style="text-align: justify;">In order to build a where clause you need 3 things. The property your testing against. The value your testing for. And the operator your applying to your test. We already have our list of known properties from earlier in our combo box. We can also easily find out the type of those properties and hence limit the operations we want the user to be able to use. Finally depending on the property type, we can choose to update the UI and allow the user to type a value in a box for a string/int, or select a radiobutton for a bool etc...</p>
<p style="text-align: justify;">once we've obtained these 3 pieces of info, we can build our clause as follows.</p>

```csharp
string _prop = "DeviceManufacturerId";    //obtained from combobox
string _value = "123";        //int value obtained through textbox
string _operator = "=="; //taken from user selected operator on UI

public static Expression GetNextExpression(ParameterExpression pe,
    string _prop, string _value, string _operator)
{
   Expression left, right;
   OperatorDelegate operatorMethd;

   Type typeOfPropery = typeof(Device) //Get Prop Type
      .GetProperty(_prop)
      .PropertyType;

   TypeConverter conv = TypeDescriptor
      .GetConverter(typeOfPropery);

   //Convert our input string to the same type as our Property
   object o = conv.ConvertFrom(_value); 

   //Left of expression is our property
   left = Expression.MakeMemberAccess(
      parameterExpression,
      typeof(Device).GetProperty(_prop)
   );
   
   //Right side is out type
   right = Expression.Constant(o, typeOfPropery);
   operatorMethd =  ops[_operator];
   return operatorMethod(left, right);
}
```

<p style="text-align: justify;">This is all equivalent to <strong><em>"device.DeviceManufacturerId == 123"</em></strong> portion of original query </p>
<h4>Putting it all together</h4>
<p style="text-align: justify;">Finally, now that we have our expression, we can iterate through the rest of the user generated search filters and join them together into one clause.<br />
and execute our lambda</p>

```csharp
foreach(var in in SomeCollectionOfUserFilters)
{
   //Conjoin existing + next expressions
   whereClauseExpression = Expression.And(
      whereClauseExpression, 
      GetNextExpression(parameterExpression ... )
   );
}

//Generate our Method Call Expression for the Lambda
MethodCallExpression whereCall = Expression.Call(
   typeof(Queryable),
   "Where",
   new Type[] { devices.ElementType },
   devices.Expression,
   Expression.Lambda<Func<Device, bool>>(
      whereClauseExpression, 
      parameterExpression
   )
);

IQueryable<Device> results = devices.Provider.CreateQuery<Device>(whereCall);
```
<p>Awesome!</p>
<p><em>~Eoin Campbell</em></p>
