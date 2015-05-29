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

'''
 var task = Task.Run(() => /* code that call the regulated API*/);
 CalibrationBlock.Post(task.FromDerived<BasePricingResponse, TResp>());
 return (TResp) await CalibrationBlock.ReceiveAsync();
 '''
