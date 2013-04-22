---
layout: post
title: "processes in node Part 2: pid to kill"
date: 2013-04-12 10:22
comments: true
categories: node process fork
---

Internally within node, every object has an id that V8 uses to keep track of these
object. In the same way, when we invoke a request to the operating system to create
a process, the operating system loads the program into memory and assigns it an id
which can be used to track the running program throughout its life cycle.

The id this process is assigned is called a pid and it can be accessed from within
node via process

{% codeblock lang:javascript %}
  var pid = process.pid;
  console.log(pid);
{% endcodeblock %}

*Fun fact*: if you open another terminal and type "kill -9 (process pid)." You will
have effectivly terminated the process in the other window by having the operating
system directly deliver a kill switch to your running node instance.

The process module is global so we don't need to require it. It gives us alot of
information about the running node instance that we can use in our programs.

Example operations include

  + exiting the program
  + changing directories/ getting directory information
  + getting the name of the directory of the start script
  + retrieving argumenst passed into the command line invocation of the script

####process events

specific to node.js is the ability to bind to occur at process events.

{% codeblock lang:javascript %}
  var runOnExit = function() {
   console.log('...exiting');
  };

  process.on('exit', function() {
    /* its important that you do not invoke
    * any asynchronous functions in here
    */
    runOnExit();
  });

{% endcodeblock %}

In the case of exit, the event loop terminates. The documentation specifically
mentions events scheduled with setTimeout. Its important to note that this applies
to all asych events as they all operate on setTimeout underneath.

*Consequently, if you need to write some pertinent information to the disc, Be sure
to use writeFileSync!!*

####process.kill

TLDR: Processes can send signals to each other and recieve signals from the ernal

Signals can be compared to events
and event listeners in javascript. kill()'s name is unfortunate since kill
is really a function for sending messages to other processes as OS signals. You will
use this function to tell other processes to go kill themeselves most of the time.

This isn't limited to node processes speaking amongst themeselves. Ever run node and then
exit it using *ctrl-c.*

For a great list of standard unix signals, check out this
[Awesome Chart](http://people.cs.pitt.edu/~alanjawi/cs449/code/shell/UnixSignals.htm)

We can orginaize them according to their default behaviors.

  + kill requests/reports
  + error reporting
  + device access notification
  + coordinating io

We can overide the default behaviors for some of these signals.


{% codeblock lang:javascript %}
  console.log("to delete => kill -9 " +process.pid);
  console.log("\n\n this is a a reinforced program, it will not easily die");

  /* SIGINT is sent to your process when you type in CTR-C.
     the operating system intercepts this command and relays
     it to your node instance. node's default response is to exit.
   */
  process.on('SIGINT', function() {
    console.log('you tried to press CTRL-C, go ahead, try again');
  });

  process.on('SIGTSTP', function() {
    console.log('LOL, nope - CTRL-Z won\'t save you!! either');
  });
  //this keeps the program from exiting
  var mainLoop = function() { setTimeout(mainLoop, 0); };
  mainLoop();
{% endcodeblock %}

A more realistic scenario would be a situation in which you need to
write some files to the disk before kicking off the exit procedures.

{% codeblock lang:javascript %}
  process.on('SIGINT', function() {
    console.log('writing to disk');
    saveData();
    return process.exit(0);
  });

{% endcodeblock %}

You cannot overwrite the kill -9 signal. This is why certain programs can ecome corrupted
when you kill-9 them. they are not given a chance to save state.

For more information about signals, I found these two guides to be excellent.

  + [Working with Unix Processes](http://www.workingwithunixprocesses.com/?utm_source=jstorimer.com&utm_medium=blog&utm_campaign=bookspage)
  + [The linux Programming Interface](http://www.amazon.com/The-Linux-Programming-Interface-Handbook/dp/1593272200/ref=sr_1_1?ie=UTF8&qid=1366589402&sr=8-1&keywords=linux+programming+interface)

Signals are a primitive pipeline for relaying messages. If you are looking to build some
form of complex communication pipeline between individual processes,
you are better off doing so over tcp. There are frameworks that facilitate
this such as [zeromq](http://www.zeromq.org/) with an
excellent [node binding available](https://github.com/JustinTulloss/zeromq.node).







