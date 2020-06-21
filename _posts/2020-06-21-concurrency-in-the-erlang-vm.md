---
layout: post
title: Concurrency In The Erlang VM
date: 2020-06-21 16:00:00 +5:30 GMT
share: y
---

[Erlang](https://www.google.com/url?q=https://codesync.global/media/how-erlang-got-its-name-concurrency-before-erlang/&sa=D&ust=1592766229543000) is often referred to as the “concurrency oriented programming language”.

How did it get this name? How can a language created in the 80s for the telecom industry help us now?

<!--break-->

Erlang or _Erlang/OTP_ (Open Telecom Platform) was built in 1986 at Ericsson to handle the growing telephone user base. It was designed with the aim of improving the development of telephony applications. It is great for building distributed, fault tolerant and highly available systems.

So, what relevance does a technology built in the 80s have for us? It turns out that all the reasons for which Erlang was built for is extremely useful for web servers today. We need applications to be fast and reliable - and we want it now! Fortunately that’s possible with Phoenix, the web framework built on top of Elixir (a modern language built on top of Erlang). It has great features like hot code swapping and [real time views](https://www.google.com/url?q=https://github.com/phoenixframework/phoenix_live_view&sa=D&ust=1592766229544000) - without JS!

How is all that possible? It’s largely due to how the language was built to be concurrent from bottom up - thanks to BEAM, the Erlang VM.

Before we go any further, let’s recap a few concepts from CS 101 - the Operating System!

## Concurrency

In order to understand why concurrency in the Erlang VM is so special, we need to understand how the operating system processes instructions/programs. 

The CPU is responsible for the working of a machine. It’s sole responsibility is to execute processes. Now a process is an isolated block of execution. They have their own memory, context and file descriptors. A process is then made up of many threads or the over used definition “_a lightweight process_”.

<img src="/public/concurrency/image2.png" class="img" />

The CPU is responsible for executing these processes in the most efficient format. It can execute processes one after the other. This is known as “sequential execution”. This is the most basic and oldest method of execution. It’s also pretty useless for us today - even the refrigerators we use today can do more!

<img src="/public/concurrency/image1.jpg" class="img" />

You might think that since your modern day computer allows you to do more than one thing at a time, the CPU is executing processes in parallel. Something like this,

<img src="/public/concurrency/image3.jpg" class="img" />

However this is far from the truth. We are quite far from true parallelism. There’s a whole [plethora of bugs](https://www.google.com/url?q=https://devblogs.microsoft.com/pfxteam/most-common-performance-issues-in-parallel-programs/&sa=D&ust=1592766229547000) that crop up when we try to go down this path.

The next best thing is concurrency. Concurrent execution means breaking up processes into multiple tiny bits and switching between them while executing. This happens so fast that it gives the user the illusion of multiple processes being executed at the same time - while cleverly circumventing the problems of parallelism.

It looks something like this,

<img src="/public/concurrency/image13.jpg" class="img" />

You can see that the CPU switches between process 1 and 2\. This is known as context switching. It is a pretty heavy task as the CPU needs to store all the information and state of a process to memory before switching - and then load it back when it needs to execute again. However we’ve worked on this problem for so many years that we’ve got insanely good at it.

_One last thing …_

The speed of program execution is dependent on the CPU clock cycles. The faster the processor, the faster your computer is. Moore’s law states that the number of transistors on an affordable CPU would double every two years. However if you’ve noticed, processors haven’t been getting that much faster in the past few years. That’s because we’ve hit a physical [bump in the road](https://www.google.com/url?q=https://www.theverge.com/2018/7/19/17590242/intel-50th-anniversary-moores-law-history-chips-processors-future&sa=D&ust=1592766229549000). This happened when we realized going much higher than 4GHz is very difficult and futile. Speed of light actually became a constraint! This is when we decided to do multi-core processing instead. We scaled horizontally, instead of vertically. We added more cores, instead of making a single core powerful.

<blockquote>
  Okay I know we’ve covered a lot of material here. But what’s relevant for you to remember is that context switching is heavy and that we’ve moved on to multi-core processing now. Let’s see what tricks Erlang has up its sleeve that makes it the concurrency oriented language.
</blockquote>

## Concurrency Models

In the previous section we deduced that concurrency is the best way forward to achieve multiprocessing. Over the years various languages have introduced different algorithms to achieve concurrency. Let’s have a look at three of the most significant ones relevant to our topic of discussion.

**1. Shared Memory Model**

    This is a commonly used model in popular programming languages like Java and C#. It involves different processes accessing the same block of memory to store and communicate with each other. It allows for context switching to be less heavier. 

    Though this sounds great in theory, it causes some unfriendly situations when two or more processes try to access the same shared memory block. It leads to situations like deadlocks. In order to overcome this, we have mutexes,locks and synchronization. However this makes the system more complex and difficult to scale. 

    <img src="/public/concurrency/image10.png" class="img" />

**2. Actor Model**

    This is the model used by Erlang and Rust to achieve concurrency. It depends on isolating processes as much as possible and reducing communication between them to message passing. Each process is known as an actor. 

    Actors communicate with each other by sending messages. Messages can be sent at any time and are non-blocking. These messages are then stored in the receiving actors mailbox. In order to read messages, actors have to perform a blocking read action.

    <img src="/public/concurrency/image9.png" class="img" />

**3. Communicating Sequential Processes**

    This is a highly efficient concurrent model used by Go. It’s similar to the actor model in that it uses message passing. However unlike in the actor model where only receiving messages is blocking, CSP requires both actions to be blocking. 

    In CSP, a separate store called the channel is used to transfer incoming and outgoing messages. Processes don’t communicate with each other directly, but with the channel layer in between.

    <img src="/public/concurrency/image6.png" class="img" />

## Actor Model

Let’s take a deeper look into the actor model.

An actor is a primitive form of concurrency. Each actor has an identity that other actors can use to send and receive messages from. When messages are received, they are stored in the actors mailbox. An actor must make an explicit attempt to read contents of the mailbox. Each actor is completely isolated from others and shares no memory between them. This completely eradicates the need for complex synchronization mechanisms or locks.

An actor can do three things when it receives a message,

**1. Create more actors**
**2. Send messages to other actors**
**3. Designate what to do with the next message**

The first two are pretty straightforward. Let’s look at it’s representation on a code level.

<img src="/public/concurrency/image4.png" class="img" />

You can see here that spawning a new process in Elixir - is exactly that. Unlike other languages which use a multithreaded approach to deal with concurrency, Elixir uses threads.

The key thing to remember is that these are Erlang threads which are much more lightweight than their OS counterparts. On average, Erlang processes are [2000 times](https://www.google.com/url?q=http://blog.plataformatec.com.br/2018/04/elixir-processes-and-this-thing-called-otp/&sa=D&ust=1592766229553000) more lightweight than OS processes. There could be hundreds of thousands of Erlang processes in a system and all would be well.

Erlang threads also have a PID attached to them. Using this PID, you can extract information about the memory it occupies, functions it’s running and much more. Using the PID, you can also send messages to processes and communicate.

Let’s take a look at the final responsibility of an actor - designation.

<img src="/public/concurrency/image12.png" class="img" />

You can see here that each process also has its own internal state. Everytime we send a message to the process, it does not restart the function with default values, rather it is able to retain information of the state of the running process. Inherently each actor is able to access it’s own set of memory, and keep track of it.

These three properties allow Erlang processes to be very lightweight. It eradicates the need for complex synchronization techniques and does not take up too much memory space for interprocess communication. All this contributes to high availability.

<blockquote>
  At this point of time, it’s a great idea to watch this [quick 4 minute video](https://www.google.com/url?q=https://www.youtube.com/watch?v%3DELwEdb_pD0k&sa=D&ust=1592766229555000)of the actor model. After all, a picture is worth a thousand words, and a video even more!  

  If you’ve got the time and want something much more in detail, [have a look at this video](https://www.google.com/url?q=https://www.youtube.com/watch?time_continue%3D60%26v%3D7erJ1DV_Tlo%26feature%3Demb_logo&sa=D&ust=1592766229556000) where the creator of the Actor Model explains all that I’ve said in such an eloquent way.
</blockquote>

## BEAM

Well after so many topics, we’ve finally come to the actual Erlang VM - BEAM!

<div class="tenor-gif-embed" data-postid="11313969" data-share-method="host" data-width="100%" data-aspect-ratio="1.537037037037037"><a href="https://tenor.com/view/beam-me-up-scotty-gif-11313969">Beam Me Up Scotty GIF</a> from <a href="https://tenor.com/search/beammeupscotty-gifs">Beammeupscotty GIFs</a></div><script type="text/javascript" async src="[https://tenor.com/embed.js](https://www.google.com/url?q=https://tenor.com/embed.js&sa=D&ust=1592766229557000)"></script>

The [BEAM VM was built](https://www.google.com/url?q=https://blog.lelonek.me/elixir-on-erlang-vm-demystified-320557d09e1f&sa=D&ust=1592766229557000) to be able to both compile Erlang/Elixir files into bytecode (.beam) files as well as to schedule Erlang processes on the CPU. This level of control gives a huge advantage to efficiently and concurrently run processes.

When the VM starts, it’s first step is to start the Scheduler. It is responsible for running each Erlang process concurrently on the CPU. Since the processes are all maintained and scheduled by BEAM, we have more control over what happens when a process fails (fault tolerance) and are able to more efficiently use memory and perform better context switching.

BEAM also takes full advantage of the hardware. It starts a Scheduler for each available core allowing processes to run freely and efficiently, while still being able to communicate with each other.

<img src="/public/concurrency/image8.png" class="img" />

BEAM allows Erlang to spawn thousands of processes and achieve high levels of concurrency. There is an awesome conversation on HackerNews about how the idea that thousands of processes lead to efficient concurrency.

<img src="/public/concurrency/image5.png" class="img" />

Have a [read](https://www.google.com/url?q=https://news.ycombinator.com/item?id%3D10028227&sa=D&ust=1592766229559000)!

## Contributing Characteristics

Stay calm, we’re almost at the end!

Erlang concurrency is largely due to the Actor model and BEAM. However, there are many other constructs that ensure stability as well. I’m going to give you small pointers that you can pick up for further reading.

**1. Supervisors**

There is a special Erlang process called the Supervisor. It’s goal is to figure out what to do when a process fails. It helps it reset to an initial value so that it can be sent for processing once again. 


**2. Fault Tolerance**

  Joe Armstrong, (late) creator of Erlang, said the infamous line, “Let it crash”. He [designed](https://www.google.com/url?q=https://thenewstack.io/why-erlang-joe-armstrongs-legacy-of-fault-tolerant-computing/&sa=D&ust=1592766229560000) the fault tolerant language not with the goal of preventing errors from happening, but to build structures that handle error scenarios. 

  <img src="/public/concurrency/image11.png" class="img" />
  
  The entire thread is heartwarming, [give it a read](https://www.google.com/url?q=https://news.ycombinator.com/item?id%3D19708379&sa=D&ust=1592766229561000)!  
    
**3. Distribution**

    Finally Erlang makes it extremely easy to build distributed systems. Each Erlang instance can act as a node in different devices and communicate just as easily as spawning processes. 
    
**4. No GIL!**

Unlike other interpreted languages, most famously Ruby and Python, Erlang does not have to worry about the Global Interpreter Lock. The GIL ensures that only one Ruby/Python thread runs per process on the CPU. This is a huge blow to concurrency. 
App servers like Passenger try to overcome this by creating multiple OS processors and run it on multiple cores. However as we saw before, OS threads are expensive to manage.

## Conclusion

Erlang is a beautifully constructed and well throughout language. It’s truly passed the test of time and is so much more relevant in recent times.