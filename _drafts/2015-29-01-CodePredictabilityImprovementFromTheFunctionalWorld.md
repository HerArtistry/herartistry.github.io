---
layout: post
title: Code Predictability Improvement from the Functional World
comments: true
tags: [C#, functional programming]
---

Various guidelines, books and blog posts talk about clean code, readability and maintainability but there is hardly any guidance on predictability. There are code predictability signs that overlap with readability but predictability goes beyond whether a method or property has a self-explanatory name. To me predictability for methods/properties include:
no hidden side effects
no nulls returned
no exceptions thrown
no AOP

**Command Query Separation (CQS)**

    int ReserveTable();

what does this code actually do? Can you predict that without inspecting the method?

    public int ReserveTable()
    {
    	//reserves a table
    	//returns the number of available tables
    }

Would you have been able to predict that by just looking at the method signature? How about now:

    public void ReserveTable();
    public int GetNumberOfAvailableTables();

CQS further improves predictability by guaranteeing that query methods are idempotent and safe. In other words, they do not have side effects and can be invoked multiple times.

**Avoid Aspect Oriented Programming**

Aspect Oriented Programming (AOP) violates the Principle of Least Astonishment (POLA). It makes it extremely difficult to predict what a piece of code is going to do and it deceives you into thinking that your code is simple and straight forward when it is not. Some AOP frameworks require the use of attributes which slightly improves predictability and are easier to discover than ones that do not. What problem do AOP framewoks actually solve? and is there a way to not have that problem in the first place? 

**Do not return nulls** This is a quick draft
Tony Hoare decribed it as his "billion dollar mistake" but what is really wrong with nulls and how is this releated to predictability? In object oriented programming, null is a time bomb that catches you by surprise unexpectedly. This is because any any refrence type can in theory be null and it can therefore blow up on your face if you do not account for it. Take for example this code:

    var order = mediator.Execute(getOrderQuery(23));

what is the value of order? Well, it is either an order object or null; hence, you would have to check for nulls before continuing the code

    if(order != null)
    {
        // process order
    }

But this could have easily been missed and it's further amplified when you get an order that has, for example, order details. Now the order is not null but it's details could be. This hurts predictability as in an ideal world when I fetch an order I expect to either get an order or not and this can be represented better using an Option or Maybe types because when you know you're getting back a Maybe you can safely predict that the returned types is either there or not.

Some functional languages do not allow you to throw exceptions but rather return an object that has an exception/error property.

PRINCIPLE OF LEAST ASTONISHMENT

Not returning NULLs and not throwing exceptions.
Initialising values in a valid state

This draft does not emphasise the lessons learned from functional programming, It only talks about how to improve predictability. I need to cover these points and show how they can be implemented in, for example, C#:

Variables should not be allowed to change their type.
Objects containing the same values should be equal by default.
Comparing objects of different types is a compile-time error.
Objects must always be initialized to a valid state. Not doing so is a compile-time error.
Once created, objects and collections must be immutable.
No nulls allowed.
Missing data or errors must be made explicit in the function signature.

NOte: this is a copy from the excellent [Is your programming language unreasonable?](http://fsharpforfunandprofit.com/posts/is-your-language-unreasonable/) article.
