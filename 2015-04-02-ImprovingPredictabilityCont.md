---
layout: post
title: Code Predictability Improvement from the Functional World
comments: true
tags: [C#, functional programming]
---
various guidelines, books and blog posts about clean code and readability but there is hardly any guidance on predictability. There are code predictability signs that overlap with readability but predictability goes beyond whether a method or property has a self-explanatory name. To me predictability for methods/properties include:
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

Would have been able to predict that by just looking at the method signature? How about now:

    public void ReserveTable();
    public int GetNumberOfAvailableTables();

CQS further improves predictability by guaranteeing that query methods are idempotent and safe. In other words, they do not have side effects and can be invoked multiple times.

**Avoid Aspect Oriented Programming**

Aspect Oriented Programming (AOP) violates the Principle of Least Astonishment (POLA). It makes it extremely difficult to predict what a piece of code is going to do and is deceives you into thinking that your code is simple and straight forward when most of time it is not.

**Do not return nulls**

 
PRINCIPLE OF LEAST ASTONISHMENT

Not returning NULLs and not throwing exceptions.
Initialising values in a valid state