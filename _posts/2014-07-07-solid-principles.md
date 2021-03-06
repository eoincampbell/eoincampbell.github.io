---
layout: post
title: SOLID Principles
date: 2014-07-07 21:06:26.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Coding
tags: []
meta:
  _edit_last: '1'
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<h2>Introduction</h2>
<p>Over the past few months Greenfinch has hired a number of new developers at varying levels of seniority. One of the go-to questions for our interview was to ask a potential candidate about SOLID principles in Object Oriented programming. Astonishingly, many candidates either didn't know what they were or only had an academic understanding of them and could not talk about them in a practical sense with regards to real projects that they'd worked on.</p>
<p>Because we have a number of development engineers spanning various levels of experience, we thought it would be appropriate to have a quick refresher course on SOLID with some practical examples in one of our lunchtime brown-bag sessions. You can find the presentation below.</p>
<h2>Presentation</h2>
<p><iframe style="text-align: center" src="https://prezi.com/embed/xvguidiyozaz/?bgcolor=ffffff&amp;lock_to_path=0&amp;autoplay=0&amp;autohide_ctrls=0&amp;features=undefined&amp;disabled_features=undefined" height="400" width="550" allowfullscreen="" frameborder="0"></iframe></p>
<h2>Object Oriented Programming Concepts</h2>
<h3>Inheritance</h3>
<p>Inheritance is when an object or class is based on another object or class, using the same implementation (inheriting from a class) or specifying implementation to maintain the same behaviour (implementing an interface). It is a mechanism for code reuse and to allow independent extensions of the original software via public classes and interfaces giving rise to a hierarchy.</p>
<p>Inheritance should not be confused with sub-typing though they can agree with one another. In general sub-typing establishes an is-a relationship, while inheritance only reuses implementation and establishes a syntactic relationship, not necessarily a semantic relationship.</p>

```csharp
public class Vehicle { ... }

public class RoadVehicle : Vehicle { ... }

public class Car : RoadVehicle { ... }

public class Truck : RoadVehicle { ... }
```

<h3>Encapsulation</h3>
<p>Encapsulation refers to the bundling of data with the methods that operate on that data. Encapsulation is used to hide the values or state of a structured data object inside a class, preventing unauthorized parties' direct access to them. Publicly accessible methods are generally provided in the class (so-called getters and setters) to access the values, and other client classes call these methods to retrieve and modify the values within the object.</p>
<p>It's important to understand that Encapsulation doesn't just mean classes are property bags with getters &amp; setters and a handful of methods. As well as hiding implementations and exposing only the APIs required for the consumers to get what they need from a class or module, it also relates to how you structure your application architecture. Properly delineating your architecture into Core, Data, Service, Façade and Consumer layers, for example, will help encapsulate the functionality below from the callers above and keep your architecture decoupled.</p>
<p>Another important consideration with regards encapsulation is testability. Too often, we'll start with a correctly encapsulated piece of code, and then when it comes time to unit test we realise that the functionality we want to test is buried inside private/inaccessible methods. This should be a red-flag to you that you need to rethink your design. Rather than just making these methods public or slapping a InternalsVisibleTo attribute on your assemblies, consider that maybe you need to abstract that functionality out of your method into another responsible class or module.</p>
<h3>Polymorphism</h3>
<p>At run time, objects of a derived class may be treated as objects of a base class in places such as method parameters and collections or arrays. Base classes may define and implement virtual methods, and derived classes can override them, which means they provide their own definition and implementation. At run-time, when client code calls the method, the CLR looks up the run-time type of the object, and invokes that override of the virtual method. Thus in your source code you can call a method on a base class, and cause a derived class' version of the method to be executed.</p>

```csharp
public class Program
{
    static void Main(string[] args)
    {

        // Polymorphism at work #1: a Rectangle, Triangle and Circle
        // can all be used whereever a Shape is expected. No cast is
        // required because an implicit conversion exists from a derived
        // class to its base class.
        List shapes = new List();
        shapes.Add(new Rectangle());
        shapes.Add(new Triangle());
        shapes.Add(new Circle());

        // Polymorphism at work #2: the virtual method Draw is
        // invoked on each of the derived classes, not the base class.
        foreach (Shape s in shapes)
        {
            s.Draw();
        }

        // Keep the console open in debug mode.
        Console.WriteLine("Press any key to exit.");
        Console.ReadKey();
    }
}

public abstract class Shape
{
    public abstract void Draw();
}

public class Circle : Shape
{
    public override void Draw()
    {
        Console.WriteLine("Drawing a circle");
    }
}
public class Rectangle : Shape
{
    public override void Draw()
    {
        Console.WriteLine("Drawing a rectangle");
    }
}
public class Triangle : Shape
{
    public override void Draw()
    {
        Console.WriteLine("Drawing a triangle");
    }
}
```

<h3>Cohesion &amp; Coupling</h3>
<p>Cohesion and coupling are worth mentioning in unison. Cohesion refers to how closely related (logically/semantically) the pieces of functionality are, that are exposed by a particular module or class. If you ask yourself the question, "Do these pieces of functionality belong together?" and the answer is "Yes!" then you have a cohesive piece of code. Coupling on the other hand refers to how tightly interlinked two totally separate modules/classes are together. The more coupling that exists in your application, the more likely that changes to one piece of functionality will have an effect (possibly an adverse effect) on another.</p>
<p>In general you should aim to write code which is highly cohesive, with low coupling.</p>
<h2>What's that smell?</h2>
<p>There are a number of things that ring out to developers as wrong when they seem them in software: Duplicated code; long methods and long branching statements; unmaintainable/brittle tests; tomes of text within method comment blocks explaining the voodoo that lies before them. We typically refer to these as Code Smells but there are also architectural smells that often times go ignored. Rigid designs that are difficult to change and manipulate; Viscous &amp; complex designs that require massive surgery to get the next square feature to fit in that round interface/inheritance hierarchy, fragile &amp; immobile designs that break when we change them and result in developers having to cut corners or possibly throw DRY out the window. (don't repeat yourself - yes I realise the irony of spelling out the acronym)</p>
<p>But what's the big deal? So maybe we need to write a little more code or perform a little bit of surgery on the architecture. That's development right!</p>
<p>Well not really. At the end of the day, change equals cost. This is particularly relevant in a SME like Greenfinch where a number of our projects are bespoke engagements with customers. That cost needs to be absorbed somewhere so it's either going to cost our customers more to get the functionality required or Greenfinch needs to absorb those costs during development. It also has a negative impact on the Team. In projects with many developers where a colleague may have to extend work that you've done, you end up putting road blocks in place for them. Overall it impacts on development/product morale and soon people are grumbling about that module or that developers code. Probably worst of all is the build up of a business debt. Some refer to this as technical debt, but really, it's the business that owns the product that is accruing these //TODO items and //MUST FIX backlog tickets that seem to grow at a faster velocity than they can be cleared.</p>
<h2>SOLID Principles</h2>
<p>SOLID is an acronym for five guiding principles to help you write better, more maintainable code. Popularized by Robert C. Martin (aka Uncle Bob) in his book Agile Software Development: Principles, Patterns, and Practices, where he gave pragmatic advice on object-oriented design and development in an agile team. SOLID stands for:</p>
<ul>
<li>Single Responsibility Principle (SRP) </li>
<li>Open-Closed Principle (OCP) </li>
<li>Liskov Substitution Principle (LSP) </li>
<li>Interface Segregation Principle (ISP) </li>
<li>Dependency Inversion Principle (DIP)</li>
</ul>
<p>Software Development is not supposed to be like a game of Jenga. You shouldn't be worried about the entire system collapsing, every time you add, remove or refactor one of the blocks of the system. These 5 principles provide guidance on how best to construct your code &amp; architecture to ensure that it's easily maintained and modified by you and your colleagues.</p>
<h3>Single Responsibility Principle</h3>
<blockquote><p>If a class has more then one responsibility, then the responsibilities become coupled. Changes to one responsibility may impair or inhibit the class’ ability to meet the others. This kind of coupling leads to fragile designs that break in unexpected ways when changed. - Robert C. Martin</p></blockquote>
<p>In a nutshell, each block of code &amp; functionality (methods &amp; classes) should be responsible for one single thing. The more things that a block of code is responsible for, the more heavily coupled it is with other pieces of functionality and behaviour, and as a result, the more likely it is to break, when you want to change just one small part of it. Let's consider a simple logging class for example.</p>

```csharp
public class EoinsLogger
{
    public enum LogTo
    {
        TheDatabase,
        TheFileSystem
    }

    public void LogMessage(string message, LogTo where)
    {
        if (where == LogTo.TheDatabase)
        {
            LogToTheDatabase(message);
        }
        else
        {
            LogToTheFileSystem(message);
        }
    }

    private void LogToTheDatabase(string message)
    {
        //ADO.NET Code

    }

    private void LogToTheFileSystem(string message)
    {
        // System.IO. Code

    }
}
```

<p>This code has a lot of different responsibilities. It's responsible for logging obviously, but it's also responsible for the decision on which underlying logging implementation to use, as well as the two specific implementation methods themselves. If another developer wants to come along and modify this, perhaps adding a third logging medium, they need to significantly alter the class in order to accomplish that. Below is a slightly better implementation. We've abstracted the actual implementations of the specific logging medium from the logger itself. Now, we've taken away some of the responsibility from the logger.</p>

```csharp
public class Logger
{
	public enum LogTo
	{
		TheDatabase,
		TheFileSystem
	}

	private ILoggerImplementation _ilog;

	public Logger(LogTo where)
	{
		if(where == LogTo.TheDatabase) _ilog = new DatabaseLogger();
		else _ilog = new FileSystemLogger();
	}
	public void LogMessage(string message)
	{
		_ilog.LogMessage(message);
	}
}

public interface ILoggerImplementation
{
	void LogMessage(string message);
}

public class DatabaseLogger : ILoggerImplementation
{
	public void LogMessage(string message)
	{
		//ADO.NET Code
	}
}

public class FileSystemLogger : ILoggerImplementation
{
	public void LogMessage(string message)
	{
		//System.IO Code
	}
}
```

<p>But it still has ownership of both the logging process, and the decision on where to log to. It breaks the OC Principle, as extending the logging to include a third implementation means modifying the brancing logic in the logger. It should be closed for extension.</p>
<h3>Open-Closed Principle (OCP)</h3>
<blockquote><p>Modules that conform to open-closed have two primary attributes: They are “Open For Extension” They are “Closed for Modification” - Robert C. Martin</p></blockquote>
<p>Let's further modify our previous logging example. It complies with our Open for Extension attribute, but it's not currently closed for modification. We can accomplish that by injecting the logger implementation to be used at runtime.</p>

```csharp
public class Logger
{
	private ILoggerImplementation _ilog;

	public Logger(ILoggerImplementation theLogger)
	{
		_ilog = theLogger;
	}
	public void LogMessage(string message)
	{
		_ilog.LogMessage(message);
	}
}

public interface ILoggerImplementation
{
	void LogMessage(string message);
}

public class DatabaseLogger : ILoggerImplementation
{
	public void LogMessage(string message)
	{
		//ADO.NET Code
	}
}

public class FileSystemLogger : ILoggerImplementation
{
	public void LogMessage(string message)
	{
		//System.IO Code
	}
}
```

<p>That's much better now. Our logger class is simply responsible for logging. Extension can be accomplished by creating new loggers. And the decision on which medium to use has been removed (for the consumer to decide upon) when it instantiates the logger.</p>
<h3>Liskov Substitution Principle (LSP)</h3>
<blockquote><p>Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program. - Some body that isn't Robert C. Martin</p></blockquote>
<p>Deciding on the correct abstractions in your architecture is important to get right and well worth having the initial design sessions on. Whiteboard it out. Decide among your architecture teams if your object hierarchy is correct. Lets look at a simple/contrived example here. Everyone learns about shapes in primary school mathematics. A rectangle is a 4 sided shape. Each side seperated by a 90 degree angle. Resulting in two long sides (the length) and two short sides (the width). You can get the area of a rectangle by multiplying its length by its width. And you can double the area of the rectangle by doubling the length of one of its sides.</p>

```csharp
public class Rectangle
{
	protected int Length { get; private set; }

	protected int Width { get; private set; }

	public Rectangle(int l, int w)
	{
		Length = l;
		Width = w;
	}

	public virtual int GetArea()
	{
		return Length * Width;
	}

	public void DoubleInArea()
	{
		Length = Length * 2;
	}
}

[TestFixture]
public class RectangleTests
{
	[Test]
	public void TestRectangeArea()
	{
		int l = 10;
		int w = 5;
		int expected = 50;
		Rectangle r = new Rectangle(l, w);
		int actual = r.GetArea();
		Assert.AreEqual(expected, actual);

		r.DoubleInArea();
		int newexpected = 100;
		int newactual = r.GetArea();
		Assert.AreEqual(newexpected, newactual);
	}
}
```

<p>We also learn that a square is just a more specialised type of rectangle where all 4 sides are equal in length to one another. So it seems pretty reasonable to design a system where a Square is just a specialised sub-type of Rectangle. Right ?</p>

```csharp
public class Square : Rectangle
{
	public Square(int side)	: base(side, side)
	{

	}

	public override int GetArea()
	{
		return Length * Length;
	}
}

[TestFixture]
public class SquareTests
{
	[Test]
	public void TestSqureArea()
	{
		int l = 10;
		int expected = 100;
		Rectangle r = new Square(l);
		int actual = r.GetArea();
		Assert.AreEqual(expected, actual);

		r.DoubleInArea();
		int newexpected = 200;
		int newactual = r.GetArea();
		Assert.AreEqual(newexpected, newactual);
	}
}
```

<p>But wait, what's happened here. The implementer of Square has overridden the GetArea() method to multiply the length by itself. A perfectly reasonable assumption in the context of a square. But the underlying type has a DoubleInArea() method which doubles the length of the Square. Calling this method in conjunction with the Square's GetArea() method doesn't just double the length. It quadruples the area. This kinda of issue rears it's head all too often in Software Development where presumptuous but naive abstractions fail in real world use.</p>
<p>So what would a better solution have been here. Maybe both rectangle and square should have implemented an IFourSidedShape interface which forced the implementer of Square to explicity implement both the GetArea &amp; DoubleInArea methods.</p>
<p>Remember if it walks like a duck, and quacks like a duck, but needs batteries, you probably have the wrong abstraction.</p>
<h3>Interface Segregation Principle (ISP)</h3>
<blockquote><p>Classes that have fat interfaces are classes whose interfaces are not cohesive. In other words, the interfaces of the class can be broken up into groups of member functions. Each group serves a different set of clients. - Robert C. Martin</p></blockquote>
<p>Here's a very simplified example of a FileSystemManager. It's singly responsible for all file I/O in our Application. It encapsulates and abstracts away the file I/O code. It's decoupled. It's cohesive in it's responsibilities.</p>

```csharp
public class FileSystemManager
{
	public void ReadFile() { }

	public void WriteFile() { }
}
```

<p>Great, but it has a pretty fat interface. Reading AND Writing files. Perhaps not every module or service that consumes this code cares about reading and writing. A logging module might only care about writing to the file system. A configuration service might only care about reading the .config files off the disk. Interface segregation is about logically splitting up the functionality of your code into smaller more semantically and logically coherent APIs for the consumers that are going to use them. In the following example, we've broken the file system manager down to implement two seperate interfaces; an IFileReader and an IFileWriter. Consumers of this code can then treat the FileSystemManager as one or the other depending on their specific needs. Furthermore, new implementations (e.g. a BlobStorageSystemManager) need only implement the interfaces that it requires.</p>

```csharp
public interface IFileReader
{
	void ReadFile();
}

public interface IFileWriter
{
	void WriteFile();
}

public class ProperFileSystemManager : IFileReader, IFileWriter
{
	public void ReadFile() { }

	public void WriteFile() { }
}

public class ProperBlobStoargeManager : IFileReader, IFileWriter
{
	public void ReadFile() { }

	public void WriteFile() {}
}
```

<h3>Dependency Inversion Principle (DIP)</h3>
<blockquote><p>A design is rigid if it cannot be easily changed. Such rigidity is due to the fact that a single change to heavily interdependent software begins a cascade of changes in dependent modules. - Robert C. Martin</p></blockquote>
<p>Dependency inversion relates to keeping our architecture decoupled. High-level modules should not depend on low-level modules. Instead both should depend on abstractions. Those abstractions should not depend on the details. Again the details should depend on abstractions. Lets look at a simple example of some hierarchical classes which have coupled dependencies on each other.</p>

```csharp
public class FacadeLayerManager
{
	private ServiceLayerManager _serviceLayerManager;

	public FacadeLayerManager ()
	{
	//Instantiate _serviceLayerManager
	}

	public List<object>GetData()
	{
		return _serviceLayerManager.GetData();
	}
}

public class ServiceLayerManager
{
	private DataManager _dataManager;
	
	public ServiceLayerManager ()	
	{
		//Instantiate _dataManager
	}
	
	public List<object>GetData()
	{
		return _dataManager.GetData();
	}
}

public class DataManager
{
	public List<object>GetData()
	{
		//Get Data From Database
	}
}
```

<p>Here we have a simple 3-tier architecture where the facade layer makes calls to a service layer to get data, and the service layer makes calls to the lower module DataManager. But this is a tightly coupled architecture. We cannot test our ServerLayerManager without creating an instance of our DataManager connecting to our database. Our higher level modules depend on the lower level rather than on an abstraction.</p>
<p>Instead, we can replace the instance fields in each layer with an abstraction (Interface) and inject our specific implementation via the constructor.</p>

```csharp

#region Interfaces

public interface IBetterFacadeLayerManager
{
	List<object>GetData();
}

public interface IBetterServiceLayerManager
{
	List<object>GetData();
}
	
public interface IBetterDataManager
{
	List<object>GetData();
}

#endregion

#region Implementations
public class FacadeLayerManager : IBetterFacadeLayerManager
{
	private IServiceLayerManager _serviceLayerManager;

	public FacadeLayerManager (IServiceLayerManager injectedManager)
	{
		_serviceLayerManager = injectedManager;
	}

	public List<object>GetData()
	{
		return _serviceLayerManager.GetData();
	}
}

public class ServiceLayerManager
{
	private IBetterDataManager _dataManager;

	public ServiceLayerManager (IBetterDataManager injectedManager)
	{
		_dataManager = injectedManager
	}
	
	public List<object>GetData()
	{
		return _dataManager.GetData();
	}
}
		
public class DataManager : IBetterDataManager
{
	public List<object>GetData()
	{
		//Get Data From Database
	}
}
```
<p>Now we've removed the tightly coupled dependencies at each layer. We can also take advantage of Dependency Injection tools and frameworks such as AutoFac, Ninject, Unity (or SpringDI in the Java world) to automatically inject the correct concrete implementation at run time based on some configuration settings.</p>
<h2>Summary</h2>
<p>OO Design is easy to get wrong. It's especially easy for a design to get out-of-control if you don't keep good principles and practices in mind at every step of development. Putting that first hack in there, or cutting that first corner is akin to thrown the pebble down the side of a snow topped mountain. It's going to turn into a very large snowball, very quickly, and once it does, it's going to be far more difficult to stop. So following good OO Principles, follow these SOLID principles, talk to one another during the design phase, think about and put the time into designing and writing your code to a high quality. It'll serve you, your company and your customers far better in the long run.</p>
<p><i>~Eoin Campbell</i></p>
