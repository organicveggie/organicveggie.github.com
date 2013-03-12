---
layout: post
title: "To Rewrite or Not to Rewrite: The Ugly Question"
date: 2010-04-24 14:07
comments: true
categories: 
 - Development
 - Management
 - Programming
 - Startups
---
I recently had a discussion about the idea of rewriting software from scratch. I actually played the devil's advocate and argued against ever throwing out and rewriting, which really got me thinking about the whole concept.
<!-- more -->
The discussion centered around article by Joel Spolsky (of [Joel on Software](http://www.joelonsoftware.com/)) titled [Things You Should Never Do, Part 1](http://www.joelonsoftware.com/articles/fog0000000069.html). His points against rewrites include:

1. The ugly code you throw out has been hardened and tested. It's filled with bug fixes. You're throwing out that knowledge and expertise.
1. You're throwing out market leadership and "giving a gift of two or three years to your competitors".
1. You're not going to do a better job writing the code a second time than you did the first time, especially since it's unlikely you have the same team that wrote the earlier version.
1. You will introduce new bugs.

Joel further argues that there are three major reasons developers want to rewrite code and none of them require rewrites:

1. Architectural problems. The "you got your gui in my business logic" problem. This can be handled by small but steady code refactorings.
1. Inefficiency. Again, can be handled by small code refactorings.
1. The code is fugly. This may be due to complexity and bug fixes, in which case see point #1 above. Or it may be due to poor and changing naming conventions, in which case it can be fixed by a simple Find-Replace.

These are all excellent points. On some level, I agree with this entirely. Even many nasty combinations of all three problems can be solved by steady refactorings. I have worked for places where people pushed for rewrites that weren't necessary. But these were larger businesses with a well established core product. These were not early startups. That's why I believe Joel makes several assumptions which are fatal to his arguments.

First, he assumes the software project is really large and complex. While some of us may have worked on projects of that size and scope, quite of few of us work on much smaller projects. Simply put, it's a matter of scale.Second, as a corollary of his first assumption, Joel also assumes that a rewrite requires years not months. Again, this is likely true for a product like Excel or Word... but this simply isn't true for many of the sites and products I've worked on. Furthermore, the use of agile or rapid development technologies such as Ruby on Rails can dramatically shrink this window.

Third, and perhaps most importantly, Joel assumes that the time required to cope with a messy code-base and make steady refactorings is significantly less than the time required to rewrite the app. And he assumes that's a worthwhile trade off. This may be clear cut for larger products or companies, but I question whether or not that's accurate for a startup. The more tangled your code, the longer it takes you to make changes. The longer it takes to make changes, the less nimble you are and the longer it takes you to respond to changes in company direction or marketplace demands.

It's that last point that I believe is most important to those of us working for small startups. We tend to be small young companies who are still striving to find our exact place in the wider world. We're often in cutting edge spaces where there is no clear cut path to success. And usually we're steadily seeing greater numbers of competitors in our space. It seems to me that agility is vitally important to people us. We need to be able to makes changes rapidly as our knowledge of the space evolves. Fundamentally, I think it's better to have a decent product/feature/whatever out in the hands of consumers than it is to have a nearly perfect product that's still under development. Don't get me wrong, I'm sure I'm preaching to the choir. :) But, I think it's critical to keep the need for agility and nimbleness in the forefront of our thoughts.

Fourth, Joel assumes any architectural problems can be solved by steady refactoring. Frankly, I disagree. I think there exist serious architectural flaws, especially related to scalability that cannot be easily solved by refactoring. eBay, LinkedIn, Facebook and Yahoo have all had major rewrites in their history that were directly attributed to serious architectural failings.

That is not to say that a full rewrite is necessarily a desirable goal. :) However, it takes careful management and planning to avoid finding yourself in this position. eBay used to employ a strategy they called [headroom](http://www.svproduct.com/engineering-wants-to-rewrite/), which basically set aside 20+% of all development time to refactor code and it keep it in top working order. While I think it may very difficult to employ such a strategy in a startup, it may be worth considering.