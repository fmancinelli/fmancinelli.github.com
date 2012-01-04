---
layout: post
title: "The biggest threat of post-industrial economy: Turing machines"
---

We are gradually transitioning towards a post-industrial economy, where manufacturing is giving way to services. A de-industrialized phase where the added value resides in the information and in its creation and exchange.

The "railway network" for this economy is the Internet, and their "steam machines" are the computers.

However this comparison is flawed. Though a computer is actually a machine, it has a trait that is missing from other, more ordinary, machines: it's *universal*.

To understand what this means, think about a washing machine and a dishwasher. They are built of very similar parts (i.e., a motor, a water pump, etc.) and have a quite similar purpose (i.e., washing), but you can't easily use a washing machine like a dishwasher and viceversa. 

This is not true for computers. The same piece of hardware can in fact be turned into almost anything we want (depending on its physical limitations), just by changing the instructions it is going to execute.

While it's very difficult to turn a pocket calculator into an alarm clock, it's quite easy to make the very same computer act like one of them.

This ability is called *[Turing completeness](http://en.wikipedia.org/wiki/Turing_completeness)*. 

A *[Turing machine](http://en.wikipedia.org/wiki/Turing_machine)* is a formal model of a machine that is able to manipulate information. This machine is made of a header that can read and write symbols on an (infinite) tape, by following some predefined logic.

This model was devised by *[Alan Turing](http://en.wikipedia.org/wiki/Alan_Turing)* in 1931, and basically provides a way for expressing everything that can be computed.

The core of a *Turing machine* is its *transition function*, a mathematical function that defines what is the next step the machine has to perform, given its current state.

It would be very painful but, in principle, we could write this function to model our pocket calculator or our alarm clock. We would end up with two different machines, one "running" with the *F<sub>Pocket calculator</sub>* transition function, and the other with *F<sub>Alarm clock</sub>*.

*Alan Turing's* great intuition was to notice that it's not necessary to have a pre-defined version of the transition function embedded in the machine. A machine, in fact, might read that transition function from the tape and then act accordingly.

The transition function of such a machine would define the logic for reading a properly encoded transition function from the tape, and then start to operate the machine as if it was bundled with the transition function that has just been read.

Such a machine is called *[universal Turing machine](http://en.wikipedia.org/wiki/Universal_Turing_machine)* and it is able to simulate all the other machines. 

This machine is the model of the computers we use every day, and that allow us to write documents, play games, make calculations and... set alarms.

Going back to our post-industrial economy, why is this a threat?

The problem is that since you have a *universal Turing machine* or, equivalently, a *Turing complete* device, you can use it to compute whatever function you want. In other words you can use it for running any program/application you want.

It turns out that that the devices we use in our everyday life are *universal Turing machines:* from the smartphone in our pocket, to the set top box connected to our TV, from the ADSL modem, to the e-book reader. 

However, post-industrial economy actors don't want their devices to be used as *universal Turing machines* because they want to control the information that can be manipulated on them.

So, your iPhone, while it's perfectly capable of running whatever operating system you might want to run on it, is limited to iOS. It could also run whatever application written for iOS, but it's limited to those available on the App Store.

Same story for an Android device or a set top box.

How can you limit a *universal Turing machine?* Simply by complicating its transition function in order to prevent the execution of unwanted instructions.

Translated to more down-to-earth terms: *digital rights management*, *spywares* and so on, running on our devices that prevent us to take full advantage of it.

But this goes far beyond the single device. We might extend this reasoning to the Internet, which is an indispensable infrastructure for the post-industrial economy and can be rightfully considered as well as a global *universal Turing machine*.

Instead of limiting what can be executed on a single device, they are limiting what a device can communicate by blocking certain protocols or routing.

Translated to more down-to-earth terms: *[China's great firewall](http://it.wikipedia.org/wiki/Great_Firewall)*, *[Stop Online Piracy Act](http://en.wikipedia.org/wiki/Stop_Online_Piracy_Act)* and so on.

*Universal Turing machines* are the foundation of our post-industrial economy but their power represents also the biggest threat to those who want to manage that economy. And they are doing all they can to cripple them and our freedom.

There is a *war on general purpose computation* that is already taking place. We should better prepare to fight in order to defend our freedom to be *universal*.

<object width="560" height="315">
<param name="movie" value="http://www.youtube.com/v/HUEvRyemKSg?version=3&amp;hl=en_US"></param>
<param name="allowFullScreen" value="true"></param>
<param name="allowscriptaccess" value="always"></param>
<embed src="http://www.youtube.com/v/HUEvRyemKSg?version=3&amp;hl=en_US" type="application/x-shockwave-flash" width="560" height="315" allowscriptaccess="always" allowfullscreen="true"></embed>
</object>

