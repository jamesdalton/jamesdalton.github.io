---
layout: post
title: "Why async is important to a web service"
description: How using async calls in a .Net app can improve your throughput.
categories: Concurrency .Net
tags: Async .Net
---

At a high level, ASP.Net uses a thread pool to process requests. How many threads is based on a number of factors including configuration and work load. New threads will be added to the pool as needed but there is a startup penalty. According to [ASP.NET Thread Usage on IIS 7.5, IIS 7.0 and IIS 6.0](https://blogs.msdn.microsoft.com/tmarq/2007/07/20/asp-net-thread-usage-on-iis-7-5-iis-7-0-and-iis-6-0/), it only adds 2 per second. If there is a sudden burst of activity, it will take time to spin up new threads. One way to avoid the penalty is to make the best use of the threads available.

One way to get the most out of a thread is to release it when the code is waiting on an external operation to complete. If the thread is released it will go back to the pool and is available to execute another task. Code that behaves this way will require fewer threads than code that doesn't.
In .Net a thread is released by using async methods. For example, when calling `HttpClient.GetAsync`, the call is made and execution continues without waiting for the response. When the method reaches a point where it must have the response to continue, it will `await` the result. It is the `await` that allows the thread to be released. Execution then returns to the method caller which in turn might `await` the result of the current method and so on up the stack. At that point the thread can be released. Once the response arrives execution picks up after the `HttpClient.GetAsync` call on the next available thread.

## Basic Example

Here is the important part of a simple command line app with Console.WriteLine calls to help demonstrate execution flow.

```csharp
static async Task<string> MakeRequest(HttpClient client)
{
    Console.WriteLine("Making get request.");
    var responseTask = client.GetAsync("http://google.com");
    Console.WriteLine("Request made waiting for result.");
    var response = await responseTask;
    Console.WriteLine("Have response, getting Content.");
    var resultTask = response.Content.ReadAsStringAsync();
    Console.WriteLine("Waiting for body.");
    var result = await resultTask;
    Console.WriteLine("Returning result.");
    return result;
}
static async Task Main(string[] args)
{
    Console.WriteLine("Enter main.");
    using (var client = new HttpClient())
    {
        Console.WriteLine("Calling MakeRequest.");
        var makeRequestTask = Program.MakeRequest(client);
        Console.WriteLine("Getting result.");
        var result = await makeRequestTask;
        Console.WriteLine("Have result");
        Console.WriteLine(result.Length);
    }
    Console.WriteLine("Exiting main.");
}
```

Running this produces the following output.

```text
$ dotnet run
Enter main.
Calling MakeRequest.
Making get request.
Request made waiting for result.
Getting result.
Have response, getting Content.
Waiting for body.
Returning result.
Have result
45962
Exiting main.
```

Line 6 of the output surprises a lot of developers. In main it comes after the call to MakeRequest but is printed before MakeRequest returns a result. That's because after line 5 execution returns to main and at line 21 `awaits` the result of MakeRequest. It is possible to wait on more than one async call at once. The next example shows how and the difference using async can make.

## Throughput example

There are two parts to this example, an API and a client. The API is simple with two endpoints, /api/sync and /api/async. Both delay the response by 0.5 - 2.0 seconds calling `Thread.Sleep` and `Task.Delay` respectively. Sleep holds the thread until the time expires while Delay release the thread and resumes after.
The client is simple and the important part is this method.

```csharp
static async Task RunTestAsync(string url, int count)
{
    List<Task<bool>> gets = new List<Task<bool>>();
    var stopWatch = new Stopwatch();
    stopWatch.Start();
    for (var i = 0; i < count; i++)
    {
        gets.Add(MakeRequestAsync(url));
    }
    var results = await Task.WhenAll(gets);
    var total = results.Count();
    var success = results.Where(x => x).Count();
    var error = results.Where(x => !x).Count();
    Console.WriteLine("Time\tTotal\tSuccess\tError");
    Console.WriteLine($"{stopWatch.ElapsedMilliseconds/1000.0} s\t{total}\t{success}\t{error}");
}
```

Line three defines a list of tasks that return a Boolean. It is used to keep track of the tasks so it will know when all have finished. Lines 6 - 9 call `MakeRequestAsync` `count` times adding it to the list. It then `awaits` at line 10 for all the requests to finish. The results variable is of type List&lt;bool&gt;, true if the request succeeded and false if it failed. Doing it this way may look a little odd, but it lets me avoid the need for locking. The framework takes care of gathering the results so all I have to do is tally them.

The results of running it produce numbers like this. The results will vary since the delay in response is random.

count|sync (s)|async (s)
---|---|---
100|12|2
250|21|2
500|38|2
750|38|2

The total time for the synchronous code grows as the number of request grow while the async code stays at 2 secs. The async code isn't faster than the sync code, it can process more requests concurrently. By releasing the thread using `Delay` the thread can be used for other tasks instead of sitting idle waiting for `Sleep` to finish.

## Summary

I will admit the comparison isn't fair. It would be rare for an API call to be purely async. However, with the exception of compute intensive APIs most request will spend some time waiting on I/O. No matter how fast that I/O is without async methods the thread is unable to do anything else while waiting. Releasing the thread frees it up to process another request. Maximizing free threads allow the service to process more request concurrently. That means consumers of the web service won't see delays or worse time outs while using the service.

The full source for both examples is available at <https://github.com/jamesdalton/why-async-is-important-to-a-web-service.>
