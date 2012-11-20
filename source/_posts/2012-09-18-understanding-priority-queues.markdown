---
layout: post
title: "understanding priority queues"
date: 2012-09-18 20:12
comments: true
categories: algoithms event-driven python
---

##Preliminaries
Node.js, twisted, eventmachine and gevent are all the rage amongst the cool kids
these days. Despite their inherent popularity, there seems to be a gap in the
understanding of what drives this particular paradigm of scalable architecture.

The event driven system of programming at its core revolves around the concept of
a central event loop running through a list of nonblocking procedures. Ok, thats
kind of a mouthful and probably doesn't mean much unless you already know how to
implement one already.

To put it more concretely, Lets flesh out the situation in more concrete terms. Imagine Sam the intern. Sam does alot of things around the office and each one takes a certain amount of time to do.

For the sake of simpicity, Sam is majoring in comparative literature so he's not going to be doing any career related work. He's the office bitch.

* making coffee -  5 min
* collate papers from the printer - 2 min + 20 seconds per page
* write annual TPS report - 20 min
* deliver mail to the desks -10 min
* call for takeout - 5min
* take out the trash - 2 min
* clean computer screens - 1 min per computer

For simplicity's sake Sam is only doing tasks within O(N) time. ie: the time it takes to perform a tasks grows linearly with the size of the task itself. Giving Sam a task where the the big O is quadratic is gonna give you a bad time. For more info on orders of growth, [wikipedia FTW](http://en.wikipedia.org/wiki/Big_O_notation).

Sam has to do all these tasks. What order though? The TPS report needs to get done by the end of the week so lets make that low priority wheras its very important that Sam get the coffee ready before the boss gets in so lets rank that a high priority. Likewise, who is likely going to notice if the computer screens haven't been wiped down or the trash taken out as long as you do it semi regularly? They get a rather low priority.

Already we see a basic operational directive for Sam the Comp Lit Major (who is very quickly going to be replaced with a small shell scripts)

1. Write down a list of things to do for the day ranked by priority
2. Start work on the highest priority task
3. When finished, goto: 2

Ah but in the real world, things aren't so simple. His boss and his coworkers throughout the day add things to the list. Thus everytime Sam gets back to his list, he needs to re-sort this list before starting work on the next highest priority task.

In node.js, we call these events and prioritising teh order of execution is the core of how an event loop works. With these intermittent events, Sam's day looks more like.

1. Write down a list of all tasks
2. Sort by priority
3. take highest priority item off the list
4. JUST DO IT!!
5. go back to 2

This is pretty much how a priority queue is more or less used.

The naive implemntation of such a priority queue (first in - highest priority out) can be done implimented using a combination of a stack with a sort performed after each insertion ranking by priority.

in pseudocode, this more or less looks like
    declare List alist;
    insert x -> alist
    sort(alist, sortByPriorityfunction)
    //pull off highest priority task
    y = pop(alist)
    do y

While conceptually simple, its a pretty inefficient way to go for  a couple of reasons. While Insertion takes place in [constant time](http://en.wikipedia.org/wiki/Time_complexity), The sort operation can vary from N logN to quadratic time depending on the sorting algorithm used. This happens on EVERY Insertion.  Alternativly, we can run the sort preceeding any pop which would optimize for cases where insertions are done more often than retrievals. Neither are suitable for anythign approaching realtime.

## Enter Heapsort

Heapsort are the glasses that go with the doctor's swank bowtie. HeapSort is Cool :).


