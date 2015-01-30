---
layout: post
title: Use IOC Containers Judiciously
comments: true
tags: [IOC, Containers, Dependency Injection, Inversion of Control]
---

IOC containers are a great way to manage dependencies and their lifetimes. They have concise, fluent syntax and help keep all dependency configurations neatly confined to the composition root and greatly simplify/eliminate writing code to handle lifetime management. Some even allow you to specify a convention and they scan the assemblies for you and automatically create the dependency graph. What's not to love about them?

I used to be a big fan of IOC containers and constantly utilised them to inject any dependency I had but recently I have come to use them cautiously. There are various reasons why I started doing this and they are not all due to IOC containers themselves. 

###Issues Related to IOC Containers###
####Magic####
I was constantly fighting against our IOC container and spending countless hours debugging and resolving issues caused by it. Granted, I did not spend enough time learning that particular IOC container I inherited but that is exactly the point. IOC containers introduce a fair bit of magic, as Greg Young calls it, and you can no longer expect developers to just hit the ground running (especially when you are hiring Graduates/Junior level developers). This could be said about most tools but IOC containers represent a bigger risk because they affect the whole project and are often abused.

####Deep Dependency Graphs####
IOC containers make it easier to abuse dependency injection and most developers jump into using them - just because everyone else is - before taking the time to learn what to inject and why. These developers often end up with deeply nested and complicated object graphs - I have seen situations where even helper classes were abstracted behind an interface and injected (IXXHelper -> XXHelperImpl). 

Imagine if you were to hand-code wiring all those dependencies you had, would you have ended up with the same deep object graph? IOC containers hide away the pain of dependency management -This is further amplified by containers that allow you to set conventions when wiring up dependencies - and sometimes you need to experience the pain in order to re-think and re-evaluate your approach/solution. Greg Young has summarised this very well in his [8 Lines of Code](http://www.infoq.com/presentations/8-lines-code-refactoring) video.

####Complexity and RAP Violation####
IOC containers tend to drive developers to violate the [Reused Abstractions Principle](http://codemanship.co.uk/parlezuml/blog/?postid=934) as you end up with abstractions that will always only have a single implementation, for example: IFoo -> Foo, that results in introducing unnecessary complexity into the code base. This falls squarely under the "developers misunderstandings" camp as IOC containers provide means to inject concrete implementations. However, it is still a problem in most systems utilising IOC containers today.

###How to Improve IOC Containers' Usage###
####TDD - No Mocking####
I used to be a Mockist, testing every class/method in complete isolation. This meant that I had a lot if abstractions in order to facilitate this kind of unit testing. These abstractions all needed to be managed, therefore, I had deep object graphs and most of my implementations suffered from construction over-injection. In order to resolve these, I ended up with more higher level (aggregate) abstractions that only deepened my object graphs. But all these levels of indirection were only needed to allow me to isolate the classes so I can test them separately, which DHH termed [Test-Induced Design Damage](http://david.heinemeierhansson.com/2014/test-induced-design-damage.html) in the "Is TDD Dead?" debate. I hardly use mocks now and tend to write higher level tests (more on that in a future post). This means I no longer have the same number of abstractions, hence, less dependencies to manage.

    Note: When testing this way you end up with less dependencies. It does not mean everything
    is now in a single class as code is still component-ized and conforms to SRP. These 
    components, however, are not dependencies, they are internal implementation details.

####Avoid Speculative Generality####
For something to be abstracted (excluding things that we don't have control over), it must be duplicated in at least two, or more, separate areas of the application. This way we have already proven that it will be reused.

Premature generalization adds extra complexity without immediate, or guaranteed ROI and is one of the reasons why we end up with complicated and convoluted object graphs. Jason Gorman states in [this](http://codemanship.co.uk/parlezuml/blog/?postid=934) post about RAP:

>Writing code on the basis that it may be applicable in multiple future scenarios is speculation, and speculative generality is considered a code smell. The danger is that when the future comes, the reusable generalisation we planned turns out to be not quite what is actually needed, or doesn't quite fit all the scenarios, or - more usually - that the future eventuality never comes up.

####Limit Your Abstractions####
I love simplicity; this is why often, dare I say it, I just do [JFHCI](http://ayende.com/blog/3545/enabling-change-by-hard-coding-everything-the-smart-way). I do not abstract away my database behind an IRepository and generally try to limit my abstractions. This leads to the avoidance of deep compositional graphs and improves simplicity as you reduce the levels of indirection, as stated by [David Wheeler](http://goo.gl/1gYGpU):

>"All problems in computer science can be solved by another level of indirection, except of course for the problem of too many indirections."

You should strive to limit those abstractions to concepts as Jimmy Bogard explains [here](http://lostechies.com/jimmybogard/2014/03/20/successful-ioc-container-usage/). Moreover, try to keep the number of abstractions small; Ayende reckons you should only have [6 - 12 abstractions in your entire application.](http://ayende.com/blog/154081/limit-your-abstractions-you-only-get-six-to-a-dozen-in-the-entire-app)

###Conclusion###
 
This does not mean I have stopped using IOC containers, I have just started using it judiciously - as the title suggests - and stopped using it as an alternative to simple good object oriented design. There is no over-reliance on IOC containers and they do not drive, or influence, my implementation or design decisions. 

Jimmy Bogard summarises good IOC usage very well:

>Ultimately, successful container usage comes down to proper OO, limiting abstractions and focusing on concepts. Composition can be achieved in many forms – often supported directly in the language, such as pattern matching or mixins – but no language has it perfect so being able to still rely on dependency injection without a lot of fuss can be extremely powerful.
