---
layout: article
title: "Aspect-Oriented Programming (AOP) in .NET Core and C# Using Autofac and DynamicProxy"
date: 2018-01-19 09:00:00 -0000
author: Carlos Blanco
categories: [.NET Core, C#]
tags: [AOP, Autofac, DynamicProxy, MacOS, Tutorial]
original_site: "Nearsoft"
original_url: "https://nearsoft.com/blog/aspect-oriented-programming-aop-in-net-core-and-c-using-autofac-and-dynamicproxy/"
canonical_url: "https://nearsoft.com/blog/aspect-oriented-programming-aop-in-net-core-and-c-using-autofac-and-dynamicproxy/"
---

Implementing Aspect-Oriented Programming (AOP) in .NET Core 2 was an eye-opener. I love the idea of transferring my .NET programming skills across all platforms, and here’s how I applied the techniques I’m used to on Windows directly on a Mac.

### A New Opportunity
Switching jobs to join Nearsoft gave me a fantastic opportunity to explore the latest developments in the .NET ecosystem. As a long-time fan of C# and its evolving features, I hadn't been working with the absolute latest versions, but I had been reading up and building exercise projects. 

At Nearsoft, I began diving into .NET Core 2 to see how I could implement familiar techniques. What immediately stood out to me was Aspect-Oriented Programming (AOP).

### What is Aspect-Oriented Programming?
AOP is a programming paradigm aimed at increasing modularity by separating cross-cutting concerns from your core business logic. Behaviors that your application needs to perform in multiple places—such as logging, transaction handling, or caching—can be added seamlessly without cluttering your actual code.

For the examples below, we will use **Autofac** and **DynamicProxy** from the Castle project.

You will need the .NET Core SDK installed on your system, and optionally, Visual Studio (VS). On a Mac, you can easily install them using Homebrew:

```bash
macbook:aop cblanco$ brew install --cask dotnet-sdk
macbook:aop cblanco$ brew install --cask visual-studio
```

### Setting Up a New Project
With .NET installed on your machine, let’s create a new console project. Open up a terminal window and run the following command:

```bash
macbook:dotnet cblanco$ dotnet new console -n aop
```

I’ll be using Visual Studio for Mac for this post, so let’s open the solution there. `cd` into the project’s folder, run the solution, and check the console output:

```bash
macbook:aop cblanco$ dotnet run
Hello World!
```

### Using Autofac and DynamicProxy
So far, so good. Now let’s implement AOP.

First, add the `Autofac.Extras.DynamicProxy` NuGet package to your solution. This package automatically brings in `Autofac` and `Castle.Core` as dependencies.

`Autofac.Extras.DynamicProxy` enables the interception of method calls on Autofac components. This matches the goal of AOP perfectly.

There are four steps to implementing interception using DynamicProxy:

 1. Create Interceptors.
 2. Register Interceptors with Autofac.  
 3. Enable Interception on Types.  
 4. Associate Interceptors with Types to be Intercepted.

We will implement two interceptors: one for logging and one for caching. Then, we will combine them to see how the whole system works together.  

### Step 1: Implementing a Logger
The first step is to implement the `Castle.DynamicProxy.IInterceptor` interface. This interceptor will log which method is being executed, the parameter values it receives, and its total execution time.

```csharp
public class Logger : IInterceptor
{
    private readonly TextWriter _writer;

    public Logger(TextWriter writer)
    {
        _writer = writer ?? throw new ArgumentNullException(nameof(writer));
    }

    public void Intercept(IInvocation invocation)
    {
        var name = $"{invocation.Method.DeclaringType}.{invocation.Method.Name}";
        var args = string.Join(", ", invocation.Arguments.Select(a => (a ?? "").ToString()));

        _writer.WriteLine($"Calling: {name}");
        _writer.WriteLine($"Args: {args}");

        var watch = System.Diagnostics.Stopwatch.StartNew();
        
        // The intercepted method is executed here.
        invocation.Proceed(); 
        
        watch.Stop();
        var executionTime = watch.ElapsedMilliseconds;

        _writer.WriteLine($"Done: result was {invocation.ReturnValue}");
        _writer.WriteLine($"Execution Time: {executionTime} ms.");
        _writer.WriteLine();
    }
}
```

### Step 2: Registering with Autofac
Next, we need to register our interceptor with Autofac in our composition root. To set the `Logger` to output to the console, we simply pass `Console.Out` as a parameter to its constructor:

```csharp
var b = new ContainerBuilder();
b.Register(i => new Logger(Console.Out));
var container = b.Build();
```

### Step 3: Enabling Interceptors
The third step is to enable interceptors on our types by calling the `EnableInterfaceInterceptors()` method during registration.

For this example, we'll use a very simple `Calculator` class and an `ICalculator` interface as our intercepted type:

```csharp
public interface ICalculator 
{
    int Add(int a, int b);
}

public class Calculator : ICalculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}
```

Modify the composition root to register this type and enable interceptions:

```csharp
var b = new ContainerBuilder();
b.Register(i => new Logger(Console.Out));
b.RegisterType<Calculator>().As<ICalculator>().EnableInterfaceInterceptors();
var container = b.Build();
```

### Step 4: Associating Interceptors
Finally, associate the interceptors with your types. You can do this right in the registration chain by calling the `InterceptedBy` method:

```csharp
b.RegisterType<Calculator>()
 .As<ICalculator>()
 .EnableInterfaceInterceptors()
 .InterceptedBy(typeof(Logger));
```

Once set up, the methods in the `Calculator` type will be intercepted by the `Logger`, outputting the following to the console:

```bash
macbook:aop cblanco$ dotnet run
Calling: aop.Domain.ICalculator.Add
Args: 5, 8
Done: result was 13
Execution Time: 0 ms.
```

### Gotchas
It's worth noting that you can also use attributes to associate interceptors with types. However, there are some caveats—for instance, you cannot use interception with private classes. You can find more details in the Autofac documentation.

### Taking It Further: Implementing a Memory Cache
We’ve successfully wrapped our business logic with a logging interceptor without intermingling the code. But we can take this further by layering interceptors to stack functionality.

Let's implement a simple memory cache. While not production-ready, it demonstrates how to combine interceptors to prevent executing an expensive method twice for the same arguments.

```csharp
public class MemoryCaching : IInterceptor
{
    private Dictionary<string, object> _cache = new Dictionary<string, object>();

    public void Intercept(IInvocation invocation)
    {
        var name = $"{invocation.Method.DeclaringType}_{invocation.Method.Name}";
        var args = string.Join(", ", invocation.Arguments.Select(a => (a ?? "").ToString()));
        var cacheKey = $"{name}|{args}";

        if (!_cache.TryGetValue(cacheKey, out object returnValue))
        {
            invocation.Proceed();
            returnValue = invocation.ReturnValue;
            _cache.Add(cacheKey, returnValue);
        }
        else
        {
            invocation.ReturnValue = returnValue;
        }
    }
}
```
*(Note: A robust implementation would need to handle the serialization of non-primitive arguments and create a proper string hash for the cache key, perhaps using Json.NET and xxHash.)*

### Layering Interceptors
Register both interceptors in your composition root and chain them onto the `Calculator` type:

```csharp
var b = new ContainerBuilder();

b.Register(i => new Logger(Console.Out));
b.Register(i => new MemoryCaching());

b.RegisterType<Calculator>()
 .As<ICalculator>()
 .EnableInterfaceInterceptors()
 .InterceptedBy(typeof(Logger))
 .InterceptedBy(typeof(MemoryCaching));

var container = b.Build();
```

Now, let's resolve the calculator and execute the `Add` method a few times:

```csharp
var calc = container.Resolve<ICalculator>();

calc.Add(5, 8);
calc.Add(5, 8);
calc.Add(6, 8);
calc.Add(6, 8);
calc.Add(5, 8);
```

(To make the execution time difference obvious, I temporarily added a 1000ms thread sleep inside the Add method). 
As the terminal output shows, the first time a method is executed with specific parameters, it takes time. Subsequent calls with the same values are instant because the cache intercepts the call and returns the stored value:

```bash
macbook:aop cblanco$ dotnet run

Calling: aop.Domain.ICalculator.Add
Args: 5, 8
Done: result was 13
Execution Time: 1006 ms.

Calling: aop.Domain.ICalculator.Add
Args: 5, 8
Done: result was 13
Execution Time: 0 ms.

Calling: aop.Domain.ICalculator.Add
Args: 6, 8
Done: result was 14
Execution Time: 1004 ms.

Calling: aop.Domain.ICalculator.Add
Args: 6, 8
Done: result was 14
Execution Time: 0 ms.

Calling: aop.Domain.ICalculator.Add
Args: 5, 8
Done: result was 13
Execution Time: 0 ms.
```

### Final Thoughts
  
We combined two interceptors to add rich functionality (logging and caching) to our app without modifying a single line of code inside the Calculator domain class. This makes our code easier to extend, maintain, and read.  

Writing this post on a Mac using Visual Studio and .NET has been a great experience. I love that Microsoft is porting its technologies to other platforms, allowing us to work with the languages we love without being tied to a single OS.
