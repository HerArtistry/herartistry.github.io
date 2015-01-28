---
layout: post
title: Use IOC Containers Judiciously
comments: true
tags: [IOC, Containers, Dependency Injection, Inversion of Control]
---

IOC containers are a great way to manage dependencies and their lifetimes. They have concise, fluent syntax and help keep all dependency configurations neatly confined to the composition root and greatly simplify/eliminate writing code to handle lifetime management. Some even allow you to specify a convention and they scan the assemblies for you and automatically create the dependency graph. What's not to love about them?

I used to be a big fan of IOC containers and constantly utilised them to inject any dependency I had but recently I have come to use them cautiously. There are various reasons why I started doing this and they are not all due to IOC containers themselves. 

I was constantly fighting against our IOC container and spending countless hours debugging and resolving issues caused by it. Granted, I did not spend enough time learning that particular IOC container I inherited but that is exactly the point. IOC containers introduce a fair bit of magic, as Greg Young calls it, and you can no longer expect developers to just hit the ground running (especially when you are hiring Graduates/Junior level developers).

I, also, changed who I do unit tests and this affected my need for IOC. I used to be a Mockist, testing every class/method in complete isolation. This meant that I had a lot if abstractions in order to facilitate this kind of unit testing. These abstractions all needed to be managed, therefore, I had deep object graphs and most of my implementations suffered from construction over-injection. In order to resolve these, I ended up with more higher level (aggregate abstractions) that only deepened my object graphs. But all these levels of indirection were only needed to allow me to isolate the classes so I can test them separately, which DHH termed [Test-Induced Design Damage](http://david.heinemeierhansson.com/2014/test-induced-design-damage.html) in the "Is TDD Dead?" debate. I hardly use mocks now and tend to write higher level tests (more on that in a future post). This means I no longer have the same number of abstractions and levels of indirection, hence, less dependencies to manage. I love simplicity, this is why often, dare I say it, I just do [JFHCI](http://ayende.com/blog/3545/enabling-change-by-hard-coding-everything-the-smart-way).

This is because IOC containers make it easier to abuse dependency injection. In my humble opinion, dependency injection is one of the most misunderstood concepts today and most developers jump into using IOC containers - just because everyone else is - before taking the time to learn it. These developers often end up with deeply nested and complicated object graphs - I have seen situations where helper classes were abstracted behind an interface and injected. Imagine if you were to hand-code wiring all those dependencies you had explicitly, would you have ended up with the same deep object graph? IOC containers hide away the pain of dependency management and, therefore, make it easier to abuse. Sometimes you need to experience the pain in order to re-think and re-evaluate your approach/solution. Greg Young has summarised this very well in his [8 Lines of Code](http://www.infoq.com/presentations/8-lines-code-refactoring) video.

Additionally, IOC containers push developers towards abstracting everything in order to make it injectable. This introduces unnecessary complexity to the code base. . This bad design is probably due to some developers misunderstanding dependency injection than IOC containers usage but 



IOC containers tend to drive to violate the [Reused Abstraction Principle](add link) as you end up with abstractions that will always only have a single implementation, for example: Foo -> IFoo. 

This does not mean I have stopped using IOC containers, I have just started using it judiciously - as the title suggests - and stopped using it as an alternative to simple good object oriented design.

IOC containers, like any other tool, can be abused leading to an overcomplicated design. But surely a tool is not bad because some developers abuse it? The problem is that IOC containers raise the bar for adoption. Developers have to learn all the idiosyncrasies of the chosen tool in order to understand how everything is wired up. This