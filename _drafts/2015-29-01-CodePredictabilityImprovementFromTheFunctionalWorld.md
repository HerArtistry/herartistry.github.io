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

PRINCIPLE OF LEAST ASTONISHMENT

Not returning NULLs and not throwing exceptions.
Initialising values in a valid state