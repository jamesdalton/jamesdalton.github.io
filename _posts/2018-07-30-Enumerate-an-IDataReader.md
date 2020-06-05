---
layout: post
title: Enumerate an IDataReader
description: Using an IDisposable with an Enumerable and how to turn IDataReader.read into an Enumerable
categories: Databases .Net
tags: 	Datareader Dispose Enumerable
excerpt_separator: <!--more-->
---

I have never liked writing this in C#. It feels out of place.

```csharp
while (reader.read())
{
    //process row
}
```

This is the way I think it should look.

```csharp
foreach(var row in reader.read())
{
    //process row
}
```

<!--more-->

Or with LINQ

```csharp
Reader.read().Select(x => processRow(x));
```

You can mimic it by doing this.

```csharp
private IEnumerable<IDataRecord> Reader(IDataReader reader)
{
    while (reader.read())
    {
        yield return reader;
    }
}
```

And then use that in a data access class like this.

```csharp
public IEnumerable<T> Get()
{
    ...
    using (var reader = cmd.ExecuteReader())
    {
        foreach (var row in Reader(reader))
        {
            yield return processRow(row);
        }
    }
}
```

This hides the annoying while loop, but it still exists. Turn Reader into an extension method and it is in only one place so I can pretend it does not exist. Even though I think the code looks better, I have shied away from it. I was not sure if the framework would properly dispose the reader.

There are several things that can happen when yield returns control to the caller. The code can throw an exception. The caller does not have to enumerate the complete list. Etc. Does the framework guarantee to call the Dispose method on the reader? I have no idea, so I asked Google. The very first hit was [Why IEnumerator<T> is IDisposable](http://der-waldgeist.blogspot.com/2010/11/why-ienumerator-is-idisposable.html). He digs into the generated code and finds that it calls the Dispose method. Trust, but verify.

I quickly coded the scenarios I cared about without a lot of work thanks to xUnit and moq. One simple class with seven test methods.

```csharp
public class TestDisposableEnumerable
{
    private Mock<IDisposable> mockReader;
    
    public TestDisposableEnumerable()
    {
        mockReader = new Mock<IDisposable>();
        mockReader.Setup(x => x.Dispose()).Verifiable();
    }

    private IEnumerable<int> IterateList()
    {
        using (var reader = mockReader.Object)
        {
            yield return 1;
            yield return 2;
            yield return 3;
        }
    }

    private IEnumerable<int> IterateAndTrow()
    {
        using (var reader = mockReader.Object)
        {
            yield return 4;
            throw new Exception();
        }
    }

    [Fact]
    public void BasicForEachDisposes()
    {
        foreach (var x in IterateList())
        {
            Console.WriteLine(x);
        }
        mockReader.Verify(x => x.Dispose());
    }

    [Fact]
    public void BasicForEachWithExceptionDisposes()
    {
        try
        {
            foreach(var x in IterateList())
            {
                throw new Exception();
            }
        }
        catch {}
        mockReader.Verify(x => x.Dispose());
    }

    [Fact]
    public void BasicForEachIteratorThrows()
    {
        try
        {
            foreach(var x in IterateAndTrow())
            {
                Console.WriteLine(x);
            }
        }
        catch { }
        mockReader.Verify(x => x.Dispose());
    }

    [Fact]
    public void BasicForEachExitEarly()
    {
        foreach(var x in IterateList())
        {
            Console.WriteLine(x);
            break;
        }
        mockReader.Verify(x => x.Dispose());
    }

    [Fact]
    public void LinqSelect()
    {
        var x = IterateList().Select(y => y).ToList();
        mockReader.Verify(z => z.Dispose());
    }

    [Fact]
    public void LinqFirstOrDefault()
    {
        var x = IterateList().FirstOrDefault();
        mockReader.Verify(y => y.Dispose());
    }

    [Fact]
    public void LinqSelectAndThrow()
    {
        try
        {
            var x = IterateAndTrow().Select(y => y).ToList();
        }
        catch { }
        mockReader.Verify(z => z.Dispose());
    }
}
```

```text
$ dotnet test
Build started, please wait...
Build completed.

Test run for /.../DisposableEnumerable.dll(.NETCoreApp,Version=v2.1)
Microsoft (R) Test Execution Command Line Tool Version 15.7.0
Copyright (c) Microsoft Corporation.  All rights reserved.

Starting test execution, please wait...
1
2
3
1
4

Total tests: 7. Passed: 7. Failed: 0. Skipped: 0.
Test Run Successful.
Test execution time: 2.1820 Seconds
```

Now with the right using statements, I know the framework guarantees the calling of reader’s Dispose method and releasing the connection. But that brings up the other problem with this method. The connection can stay open longer this way depending on the calling method. Retrieving the records and then processing them will result in the connection closing sooner. Processing the records one at a time and releasing them will result in less memory used, but the connection will be open longer. 

The best choice will depend on the situation. If it is only a few rows, but the process takes a lot of time then retrieve them all and release the connection.  If it is a batch process and a substantial number of rows, then process them one at a time. Better yet use a parallel foreach and let the framework decide how many threads to use to process them concurrently. Both will keep the connection open longer with the parallel foreach finishing sooner. Most cases will fall in between the two extremes and measuring the performance with a real-world workload will determine if there is a difference between the two. 

My next post or two are going to be on databases and async calls. I am going to test out several ways to process data faster or at least provide the perception of processing faster. Knowing that it is safe to use yield to enumerate an IDataReader I can move forward with my ideas and not worry about leaking connections.

The full source is on GitHub at <https://github.com/jamesdalton/enumerate-an-idatareader>.
