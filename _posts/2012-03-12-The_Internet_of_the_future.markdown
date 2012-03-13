---
layout: post
title: The Internet of the future
---

One of the underlying idea that seems to be quite widespread is that Internet is free (as in freedom), democratic and not subject to any control. Proof is that in many recent "revolutions", namely the ones known as the Arab Spring, the Internet and the social networks running on top of it have played a very important role, guaranteeing fast communication and mob organization.

We can find this kind of dynamics even in less bloody contexts, such as protest organization and activism coordination (e.g., the recent protests against ACTA, SOPA, and so on)

This is good news but...

Internet is, as the word suggests, a system of interconnected networks. It's a giant [graph](http://en.wikipedia.org/wiki/Graph_(mathematics) connecting hosts via all kind of intermediary machinery such as routers, and so on.

This is how a small portion of the internet looks like:

[![The Internet as a graph](/images/internet.png)](http://en.wikipedia.org/wiki/Network_mapping)

If we don't consider the end-user terminals such as, for example, the laptop or the mobile phone you're using for reading this post, we can see that the nodes in that graph are *autonomous systems*.

An *autonomous system* is "a collection of connected Internet Protocol (IP) routing prefixes under the control of one or more network operators that presents a common, clearly defined routing policy to the Internet." ([Wikipedia](http://en.wikipedia.org/wiki/Autonomous_system_(Internet))

Your *Internet Service Provider* is an autonomous system, but also Universities or the entity that coordinates Research Centers' network.

In 2010 there were almost 35000 connected *autonomous systems*.

If you want to have an idea of what kind of *autonomous systems* are registered on the *Internet*, you can have a look at [this list](http://as-rank.caida.org/?mode0=as-dump-info) compiled by [CAIDA](http://www.caida.org), the *Cooperative Association for Internet Data Analysis*.

Now what is the problem with this? The problem is what happened in Egypt during the Arab Spring: at some point they shut down the *Internet*. Literally.

How was this possible? Order to the major Egyptian service provider to cut the connection to other *autonomous systems* and nobody will be able to send or receive a packet from the *Internet*. Order to the major service providers to stop routing packets from within the *autonomous system* and nobody will be able to send or receive a packet from his neighbor. Simple like that.

This is something that could happen everywhere, because *Internet* works in the same way in Egypt, in the US, in Europe and also in China.

So the assumption that the *Internet* is not subject to control is false. Obama might switch off Facebook and Google in a matter of seconds if he wants to. It's unlikely that he will do something like that, but technically he can.

What should an *Internet* that is really free (as in freedom) and not subject to control look like?

In my opinion it will be something that resembles to the network that connected old [BBSes](http://en.wikipedia.org/wiki/Bulletin_board_system) using the phone network. This network was quite widespread during the '80s and was called [FIDONet](http://en.wikipedia.org/wiki/FidoNet).

Basically, it was a network of *BBSes* that were storing and forwarding messages at each connection. A message sent from a node, could travel for days (because *BBSes* were exchanging data only few times per day) before reaching the destination. Imagine if an email takes a week to be answered which, by the way, was the case at that time on *FIDONet*.

The advantage of this system was that it couldn't be brought down, because bringing it down would mean to shut down the telephone network, which is quite more complicated than rewriting some [BGP](http://en.wikipedia.org/wiki/Border_Gateway_Protocol) tables.

Of course *BBSes* could have been shut down as well, but message routing could have been reconfigured accordingly.

Today, however, we are in an even more interesting scenario. We don't even need the telephone network for doing something like this. We have wireless access points that can replace telephone land lines. By using wireless technology, we can build a mesh of communicating devices that are able to exchange information like *BBSes* connected through *FIDONet* were able to, more than 30 years ago. 

Peer to peer protocols could automate a bit the job of correctly routing information to the good destination (remember [Gnutella](http://en.wikipedia.org/wiki/Gnutella)?)

Of course, you can forget what the *Internet* of today offers us: immediate reply to every request, interactive applications, huge data transfer.

The free (as in freedom) *Internet* of the future could be developed as a wireless mesh of interconnected networking devices that are able to route information similarly to how it is done today on the *Internet*, but with no central authority capable of shutting it down.

In fact, and luckily for us, electromagnetic waves are way more elusive than *BGP* tables.

This would truly be a free (as in freedom) and democratic *Internet*... And maybe will also solve the information overload we are experiencing nowadays.

  
