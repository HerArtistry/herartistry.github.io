---
layout: post
title: Lessons Learned
comments: true
tags: [teams, Agile]
---

As a freelancer, I have worked with numerous teams, different cultures, distinct personalities and varying experience levels and in different capacities including a developer, technical lead and architect. This presented a range of challenges when trying to maintain a good harmony in the team and delivering value to customers. Below are a number of things that have worked well for me: 

##### Good arhictectures make it harder to do the wrong thing
I have seen the effects of violating this principle over and over again. If you make it really trivial and cheap to follow patterns that you want to move away from, then you increase the liklihood of good principle not being adhered to. Hence, make it cheaper to follow good principles but more expensive to follow bad ones. It is not a silver bullet though and you'd have to be pragmatic when applying it. Consider ROI and be aware of trade-offs

##### Show don't just tell - Agreeing theories mean nothing
Under pressure to deliver, people tend to revert to type, jump back to their comfort zone and yield to the temptation of continuing with what they are familiar with. Therefore, if they are not 100% on board with the proposed changes to the architecture, if they cannot see the benefits and if they don't wholeheartedly buy into them, they won't adopt them when the going gets tough. Therefore, spending time to evangalise the theories, showing practical examples and demonstrating the advantages is paramount. What we are going to start trialling soon is weekly team only brown bag like sessions to ensure everyone in the team is on the same page in terms of direction, ways of working, architecture, etc...

##### Keep an open mind
Discussions would go a lot smoother if participants just simply try to keep an open mind. Nothing is worse than reaching an impasse due to stuborness and ego getting in the way. Shutting people down or dismissing their ideas without a convincing reason is extremely disheartening. This is very hard to cultivate but try to foster healthy debate within the team. Try to provide an environment where everyone's voice is heard and everyone's voice counts. Use the right language when facilitating, prevent sarcasm and ridicule and give people the confidence to participate.

##### Agree/disagree but commit to team decisions
Commit to team decisions even if you disagree with them. Keep your concerns within the team but do not undermine them when communicating outwardly. Undermining the team, backtracking or quickly revealing disagreements only makes it harder to sell team decisions and decreases the trust within the team. 

##### Keep it simple
Start with the simplest thing that delivers the most customer value. Then learn from it, refactor it, invest in it and improve it if needed. Developers seem to lose focus of the big picture and over-engineer or over complicate their solutions. Prove that the solution is useful before you try and "refine" it. 

##### Don't hinder refactoring
Refactoring by definition is altering code without changing behaviour. If the behaviour of the system does not change, then why should any tests fail? Afterall, the system is still behaving as expected, isn't it? If your tests go red while refactoring, then you're testing implementation details. This would make refactoring very taxing as you have to constantly context switch between implementation and fighting failing tests. Additionally, if you are changing your code and your tests at the ame time, what guarantees do you have that you have not broken anything. Your safety net is gone. 

##### Avoid magic
If you introduce new technology, tools, libraries to your system, make sure you know how they work and you can support them - ensure there's no magic. If they break, your system breaks.

##### #NoEstimates
No estimates work. You are likely to get a lot of resistence from the business - especially at first - but everyone cares about providing value to customers and if you can demonstrate how NoEstimates help cut on time wasted needlessly guesstimating on minute details they will come around. I will write a blog post about this to get into more details.

##### No meetings for meetings' sake
Evaluate the meetings you have and gauge whether they add value. If not, then don't be afraid to cancel them - that time could be spent doing something useful instead. This is true for agile ceremonies too. Just because it is useful to someone somewhere doesn't make it useful to you. 
