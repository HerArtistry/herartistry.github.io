---
layout: post
title: Transparency Is Key - A Case Study
comments: true
tags: [Architecture, Microservices, SOA, Agile]
---

Agile practices preach transprancy within and outside a team. It build trust, empowers and motivates the team. In my exprience there is nothing more deflating and morale zapping to a team than lack of transparency and trust. In this post, I will present my views on a recent project that I worked on than I considered a failure, why I think it failed and lessons learned.

Around a year ago, I joined a company to work on a SaaS system case management system. This was a re-write of the company's flagship product as it has become a monumental piece of unmaintainable, inconrehinsible code and architecture. This meant the company could neither react to market changes nor bug fixes and updates. They resorted to actually asking customers to give them access to their production environements in order to "fix the issues". I have not seen the code myself but the tales of the ingenious hacks were almost legendary in the office. Additionally, new releases had to be done over the weekend and it took around two days to release a version of the software and it entailed numerous untraceable untestable manual steps. Furthermore, releases were generally buggy as the software had no kind of automated tests and was just tossed over to the QAs before release.

Needless to say, a complete re-write was unavoidable. But a mere code revision would likely be a long and winding road back to square one. What was really needed was an overhaul of the software practices and they way the business viewed software development, release management, product management, marketing, sales, etc. Thankfully, the business came to the same conclusion and started a recruitement drive to get the right people in. There were a lot of teething issues at the start but progress was slowly being made. 

I joined the company shortly after it started embracing agile so the developers that joined before me set the stage and commenced using Continuous Integration, Pair Programming, TDD, BDD as well as all the Scrum ceremonies. 
