---
layout: post
title: "Proceses in node part 1: introduction to processes"
date: 2013-04-11 14:26
comments: true
categories: node javascript systems process
---

As a language made to exist within the browser, javascript did not originally
come with a way to do process process related tasks. Until Chrome, browser based
javascript was running within the same process as the browser.

In Chrome, every tab runs in its own process. Consequently, your javascript on any given
page runs in its own process. Now that node has brought javascript into the server and far from
the restrictive confines of the browser, We can do intersting things like run other programs from node.

###What is a process?

Much like the relationship between a Constructor function + its prototypye and the
object created by invoking the *new* keyword, a process is an operating system level invocation
of some program.

###Process vs Program - a very high level overview

Imagine the hardrive that powers your computer. Stored on it is a long range of numbers.
For all practical purposes, we can think of it as a long serial stream of bits layed out. with information
encoded on them. At some magical location is the boot sector. When the computer first starts running,
it startsloading binary data from the hardrive into the RAM and then loads this data into the cpu.
At this point we load the master process which we call the operating system kernal.
The kernal then takes care of managing other processes in the system.

At this point, we get to your program. If we were doing this in *C*, the the compiler would compile your
source code into binary code which follows an executable format. The actual encoding of this binary instruction
stream varies across cpu architectures and operating systems. They do happen to share some
common characteristics. Most of this information is located in the header of the file as information that tells
the operating system that this code is executable and information on how it is to be run.

#####Entry Point Address

At some point in the binary stream of bytes that is your program, there exists the first instruction that
needs to be loaded into the cpu. The Operating System loads this address into memory and queues it up
for running into the cpu.

#####Data

Constants in the process are stored in the data stream in some area where they can be accessed by the
Process as it runs. This includes mathematical constants such as Math.PI or string error messages.

#####Symbol and Lookup Tables.

To properly explain symbol and lookup table, I need to elaborate on what is going on at the
instruction level on your computer. Basicly, it stores the locations of all the variables and function
entry points. in memory.


###Processes verses threads

The simple $2 answer is that processes have their own copy of all the data and symbol information. In some languages, multiple threads exist within a process and all share the same data. more importantly, you cannot create threads in node.

###Processes and node

For us nodesters, the picture gets a bit more complicated. The computer loads up the program instructions
we affectionatly know as node and loads it into memory. From there, the node program as a static set of
bits on the hardrive becomes an almost living thing that loads your javascript into memory and run it!

To clarify, the process running when you run your node program is an instance of the node program. You can
have several node instances running in memory. They can even all serve http requests as long as they are
not trying to bind to the same port.

Understanding processes is incredibly useful. In Ruby, processes forking is used heavily in the design of
[unicorn](http://unicorn.bogomips.org/). Services like nodejitsu and heroku utilize smart people who understand
how processes work to architect systems that run and manage your code on the cloud. More importantly, node code
can only run on one processor at any given time but by using features such as *fork*, you can set up a master
node process that delegates tasks to subprocesses it spawns yourself. Since node processes are so
lightweight, you could concievably run hundreds on your system at the same time.





