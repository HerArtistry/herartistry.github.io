---
layout: post
title: Throttling Simultaneous Requests using TPL Dataflow
comments: true
tags: [C#, async, await, TPL, Dataflow]
---

I used the chance to refactor some legacy code to utilize and learn a bit more about the .Net's TPL. The code in question was written a few years ago and, although it does what is intended, it was overly complicated. This was part of a major refactoring of a multi-threaded service with a heavy use of callbacks and auto/manual reset events and timeouts. We recognized that this service could be made hugely simpler by moving it to an Async/Await model and in this post I will try to explain how we refactored a specific bit of code that others might find useful.

>Disclaimer: I am not a TPL expert. This refactoring was actually the first time I've used it outside a trivial Web API controller. This is simply a summary of what I learned whilst refactoring this piece of code.

The await keyword allows you to execute a piece of code asynchronously and it frees the thread while that code is being executed. It, however, does not give you control over when the asynchronous call is made and how many concurrent class are made. This is absolutely fine for most systems but sometimes there are resource intensive calls that must be regulated. The latter is our case and the number of allowed concurrent requests is determined by the owner of this service we consume (we set it in the config file). Thus, it is imperative that we never exceed this number when consuming this particular service.

In order to achieve this, we need a gateway that keeps track of the number of requests, queues them up and throttles them according to the maximum number allowed. This gateway wraps the call to the regulated API and has a static Dataflow Block that requests are posted to. This block is static, and therefore shared by all consumers of the gateway. When a request for the regulated api is received, this is posted to the queue and awaited as shown below:

    var task = Task.Run(() => /* code that call the regulated API*/);
    CalibrationBlock.Post(task);
    return await CalibrationBlock.ReceiveAsync();

One problem I encountered was dealing with the generic return type of the regulated API. The static block should handle all return types as we need to regulate all request - not group them by return type. Hence, the Task<DerivedType> should be stored as Task<BaseType>. I found the easiest way to do this is by using the below extension method, which I found in [this](http://stackoverflow.com/a/15530315/2962640) StackOverflow answer:

    public static Task<TBase> FromDerived<TBase, TDerived>(this Task<TDerived> task) where TDerived : TBase
    {
            var baseTypeTask = new TaskCompletionSource<TBase>();

            task.ContinueWith(derivedTask => baseTypeTask.SetResult(derivedTask.Result), TaskContinuationOptions.OnlyOnRanToCompletion);
            task.ContinueWith(derivedTask => baseTypeTask.SetException(derivedTask.Exception.InnerExceptions), TaskContinuationOptions.OnlyOnFaulted);
            task.ContinueWith(derivedTask => baseTypeTask.SetCanceled(), TaskContinuationOptions.OnlyOnCanceled);

            return baseTypeTask.Task;
    }

So queuing up the task now becomes:

    var task = Task.Run(() => /* code that call the regulated API*/);
    CalibrationBlock.Post(task.FromDerived<BaseResponse, TResponse>());
    return (TResponse) await CalibrationBlock.ReceiveAsync();

But how do we ensure we adhere to the allowed number of simultaneous requests? Well, the Transform Block has an options to set that when it is initialized:

    new ExecutionDataflowBlockOptions { MaxDegreeOfParallelism = MaxAllowedNumberOfSimultaneousRequests }

This will allows tasks to be queued up whlst only executing the maximum allowed simultaneous ones at a time, executing more from the queue whenever a task is complete. However, the transform block returns results in the order they were added as shown in the example below:

    class Program
    {
        static void Main(string[] args)
        {
            TestMain();
            Console.ReadLine();
        }

        public class TaskInfo
        {
            public TaskInfo(string name, int delay)
            {
                Name = name;
                Delay = delay;
            }
            public string Name;
            public int Delay;
        }

        public class TestClass
        {
            private static readonly TransformBlock<TaskInfo, string> Downloader;

            static TestClass()
            {
                Downloader = new TransformBlock<TaskInfo, string>(
               taskInfo => CreateTask(taskInfo),
               new ExecutionDataflowBlockOptions { MaxDegreeOfParallelism = 2 }
               );
            }

            public async Task<string> Calibrate(TaskInfo taskInfo)
            {
                Downloader.Post(taskInfo);
                return await Downloader.ReceiveAsync();
            }
        }

        private static async void TestMain()
        {
            _tasks = new List<Task<string>>();

            var t1 = new TestClass().Calibrate(new TaskInfo("test1", 10000));
            var t2 = new TestClass().Calibrate(new TaskInfo("test2", 2000));
            var t3 = new TestClass().Calibrate(new TaskInfo("test3", 2000));
            var t4 = new TestClass().Calibrate(new TaskInfo("test4", 2000));

            _tasks.Add(t1);
            _tasks.Add(t2);
            _tasks.Add(t3);
            _tasks.Add(t4);

            while (_tasks.Count > 0)
            {
                var firstTask = await Task.WhenAny(_tasks);
                _tasks.Remove(firstTask);

                var t = await firstTask;
                Console.WriteLine(t);
            }
        }

        private static List<Task<string>> _tasks;

        private static async Task<string> CreateTask(TaskInfo info)
        {
            Console.WriteLine("** Started processing: {0}", info.Name);
            
            await Task.Delay(info.Delay);
            
            Console.WriteLine("%% Completed processing: {0}", info.Name);
            
            return string.Format("Returned awaitable value from: {0}", info.Name);
        }
    }    
        
        // Output:
        //** Started processing: test1
        //** Started processing: test2
        //%% Completed processing: test2
        //** Started processing: test3
        //%% Completed processing: test3
        //** Started processing: test4
        //%% Completed processing: test4 
        //%% Completed processing: test1 
        //Returned awaitable value from: test1 
        //Returned awaitable value from: test2 
        //Returned awaitable value from: test3 
        //Returned awaitable value from: test4

The maximum degree of parallelism is set to 2, therefore, tasks 1 and 2 are processed in parallel. Since task 1 takes a long time, only one extra task is processed; hence, keeping within the maximum allowed simultaneous requests. However, although tasks 2,3 and 4 are done, they are not processed as the Transform Block is waiting for task 1 to finish. This is confirmed by the documentation:

>When you specify a maximum degree of parallelism that is larger than 1, multiple messages are processed simultaneously, and therefore, messages might not be processed in the order in which they are received. The order in which the messages are output from the block will, however, be correctly ordered.

Thankfully, this can be fixed by creating a custom dataflow block as shown in [this](http://stackoverflow.com/a/22894184/2962640) Stackoverflow answer. Once we add this to our test code by replacng the test class with this

    public class TestClass
    {
            private static readonly IPropagatorBlock<TaskInfo, string> Downloader;

            static TestClass()
            {
                Downloader = CreateUnorderedTransformBlock<TaskInfo, string>(
                taskInfo => CreateTask(taskInfo),
                new ExecutionDataflowBlockOptions { MaxDegreeOfParallelism = 2 }
                );
            }

            public async Task<string> Calibrate(TaskInfo taskInfo)
            {
                Downloader.Post(taskInfo);
                return await Downloader.ReceiveAsync();
            }
        }

        public static IPropagatorBlock<TInput, TOutput> CreateUnorderedTransformBlock<TInput, TOutput>(Func<TInput, Task<TOutput>> func, ExecutionDataflowBlockOptions options)
        {
            var buffer = new BufferBlock<TOutput>(options);
            var action = new ActionBlock<TInput>(
                async input =>
                {
                    var output = func(input);
                    await buffer.SendAsync(await output);
                }, options);

            action.Completion.ContinueWith(
                t =>
                {
                    IDataflowBlock castedBuffer = buffer;

                    if (t.IsFaulted)
                    {
                        castedBuffer.Fault(t.Exception);
                    }
                    else if (t.IsCanceled)
                    {
                        // do nothing: both blocks share options,
                        // which means they also share CancellationToken
                    }
                    else
                    {
                        castedBuffer.Complete();
                    }
                });

            return DataflowBlock.Encapsulate(action, buffer);
        }

Running the same code again produces the following output:

    ** Started processing: test1
    ** Started processing: test2
    %% Completed processing: test2
    ** Started processing: test3
    Returned awaitable value from: test2 // returned as soon as method was completed
    %% Completed processing: test3
    Returned awaitable value from: test3 // returned as soon as method was completed
    ** Started processing: test4
    %% Completed processing: test4
    Returned awaitable value from: test4
    %% Completed processing: test1
    Returned awaitable value from: test1

If any of the tasks in the TransformBlock throw an exception, the bloack will neither process the pending tasks nor accept any new ones. In order to prevent this, the transformation function should no throw exceptions but rather catch them and return them as part of the result. Hence, this code:

     Downloader = CreateUnorderedTransformBlock<TaskInfo, string>(
                taskInfo => CreateTask(taskInfo), // this is the transformation function
                new ExecutionDataflowBlockOptions { MaxDegreeOfParallelism = 2 }
     );

Should become:

     Downloader = CreateUnorderedTransformBlock<TaskInfo, TransformResult<string>>(
                    async taskInfo =>
                    {
                        try
                        {

                            var result = await CreateTask(taskInfo);
                            return new TransformResult<string> {Result = result};
                        }
                        catch (AggregateException ae)
                        {

                            return new TransformResult<string> {Error = ExceptionDispatchInfo.Capture(ae.InnerException)};
                        }

                    },
                new ExecutionDataflowBlockOptions { MaxDegreeOfParallelism = 2 }
        );
        
        public class TransformResult<T>
        {
            public T Result { get; set; }
            public ExceptionDispatchInfo Error { get; set; }
        }

It might not be immediately obvious but there is big problem with the above code. The problem appears when we try to track which task received which output by spitting out the task name from both the proessed task and the awaiting task. The output in this case is:

    ** Started processing: test1
    ** Started processing: test2
    %% Completed processing: test2
    ** Started processing: test3
    Returned awaitable value from: test2 *** t1 - took: 00:00:02.1086100 // t1 was waitng for test1 not test2
    %% Completed processing: test3
    ** Started processing: test4
    Returned awaitable value from: test3 *** t2 - took: 00:00:04.0231902 // t2 was waiting for test2 not test3
    %% Completed processing: test4
    Returned awaitable value from: test4 *** t3 - took: 00:00:05.9262377
    %% Completed processing: test1
    Returned awaitable value from: test1 *** t4 - took: 00:00:09.72 //t4 was waiting for test4 but got the output of test1
    All Done. Total time taken is 11 seconds

Notice how task t1 received the output from test2 instead of test1 and t2 from test3 and t3 from test4 and t4 from test1. Not a single task got the outcome it was waiting for. This is because we are no longer maintaining the order of the tasks in the TransformBlock, however, ReceiveAsync still respects that order and first ReceiveAsync called will get the first result. In the above example, t1 invoked the ReceiveAsync first but since t1's task takes longer than t2's, t2 will finish first and t1 will get the output from t2's. This has a knock on effect and each task will then receive the next task's output instead of its own.

How can this be fixed? Well, one way is to reuse the TransformBlock instead. This maintains the ordering, thus, ensuring the correct tasks are returned.

    ** Started processing: test1
    ** Started processing: test2
    %% Completed processing: test2
    ** Started processing: test3
    %% Completed processing: test3
    ** Started processing: test4
    %% Completed processing: test4
    %% Completed processing: test1
    Returned awaitable value from: test1 *** t1 - took: 00:00:10.0334752
    Returned awaitable value from: test3 *** t3 - took: 00:00:09.8336984
    Returned awaitable value from: test2 *** t2 - took: 00:00:09.9426121
    Returned awaitable value from: test4 *** t4 - took: 00:00:09.7355575
    All Done. Total time taken is 28 seconds

**************************** Re-write this draft
This solution fixed the problem and we can stop here and call it a day. However, this is still not an acceptable solution. First notice the time it now takes to complete the tasks. It has almost tripled from 11 seconds to 28 seconds. This is because the TransformBlock is waiting for the first task to finish before returning the result and since the first task is the longest, all the other tasks are queued up and waiting for it to finish. This is detrimental to performance as requests through this gateway will be affected by unrelated request; thus, long running requests will affect all other requests in the pipeline.
************************************

The ideal solutions would be to return tasks as they are completed but somehow map them to the correct consumer. Dataflow allows you to link blocks together and this is what we are going to use. Instead of all consumers awaiting for tasks on the same output queue, we can create a queue for every consumer. This way we can ensure that the correct consumers get the correct output without having to rely on the order. The 'LinkTo' method in the IPropagatorBlock allows configuring a perdicate that is used to filter the message to pass on to the linked block. We can use any identifier as long as it is always there so that we can deal with routing exceptions as well.

    async Task<TransformResult<string>> ReceiveAsync(string identifier)
            {
                var block = new BufferBlock<TransformResult<string>>();

                using (Downloader.LinkTo(
                    block, 
                    new DataflowLinkOptions {MaxMessages = 1, PropagateCompletion = true}, 
                    output => output.Id == identifier)) // this is the predicate used for filtering
                {
                    return await block.ReceiveAsync();
                }
            }

Now we need to replace the code that calls the ReceiveAsync on the static unordered TransformBlock with a call to the above method, passing in the identifier (task name):

                Downloader.Post(taskInfo);

                var outcome = await ReceiveAsync(taskInfo.Name);

                if (outcome.Error != null)
                    return "an error has occured in " + taskInfo.Name;

                return outcome.Result;

The output after making the above changes is:

    ** Started processing: test1
    ** Started processing: test2
    %% Completed processing: test2
    ** Started processing: test3
    Returned awaitable value from: test2 *** t2 - took: 00:00:02.0095554
    %% Completed processing: test3
    ** Started processing: test4
    Returned awaitable value from: test3 *** t3 - took: 00:00:03.9181103
    %% Completed processing: test4
    Returned awaitable value from: test4 *** t4 - took: 00:00:05.8321348
    %% Completed processing: test1
    Returned awaitable value from: test1 *** t1 - took: 00:00:10.0315769
    All Done. Total time taken is 15 seconds

As can be seen above, the outputs are matched to the correct tasks and there's a significant improvement in terms of time taken when compared with the TransformBlock.
