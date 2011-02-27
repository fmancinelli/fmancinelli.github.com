---
layout: post
title: Look who is tracking your footprints on the web
---

*As a new year's resolution I've decided to start (and keep updated) a blog. So here we go!*

I mostly use [Twitter](http://www.twitter.com) as a way for harvesting interesting links from my contacts: I skim trough all the unread tweets, and if I find a link that could be interesting I open it in a browser tab. Then I start reading and bookmarking what is really interesting.

However, bookmarks are not enough: when I look at my bookmark list with page titles I can hardly remember what the pages were about. Plain bookmarks, in fact, don't provide contextual information (e.g., why have I bookmarked this page? What did I find interesting in it?). So I decided to test this service, [Diigo](http://www.diigo.com), that allowed me add that contextual information to web pages (i.e., by using text highlighting and annotations).

To use Diigo I had to install an extension for my [Chromium](http://www.chromium.org) browser. Once installed I tried to annotate an already opened page but, surprisingly, the extension complained saying that "the tab was opened before installing the extension" and suggested to reload the page. Why an already loaded page isn't good for being annotated? This made a bell ring in my head... 

![Error message](/images/diigo_error.png)

Using [Wireshark](http://www.wireshark.org), a network protocol analyzer, I started looking at the HTTP traffic on my network card.

I soon discovered that some HTTP connections where made to the address `216.237.119.210` as soon as I clicked on the Diigo button in the Chromium toolbar. By further restricting the packet capture to this address I was able to isolate what was being sent by the Diigo extension... And obviously I found that all the created annotations were sent "home".

What it was not as much obvious was that the Diigo extension was sending "home" information about every site opened in a tab/window, despite the fact that I was willing to use Diigo on those pages.

The next figure shows the traffic generated when I opened [Xkcd](http://www.xkcd.org) and [Slashdot](http://www.slashdot.org). Several POST requests were made to `216.237.119.210`, using the `/.../cmd=bm_loadBookmark/` URI. The payload contained the information about the visited sites.

[![Wireshark](/images/wireshark_small.png)](/images/wireshark.png)

This is not new. A lot of services are tracking your footprints while on the web. [Facebook](http://www.facebook.com), for example, tracks every link you click on its user interface. What is different with Diigo, and with browser extensions in general, is that this tracking is more pervasive. On Facebook I can do some kind of "conscious click" because I know that I will be tracked on that link. The Diigo extension tracks everything, no matter what.

Actually there is a technical requirement for doing that. The Diigo extension automatically loads annotation information for a page and displays them as soon as the page is loaded. That's why, after opening a page, a call to the Diigo service is done. Nevertheless this should not be the default behavior: I don't want to disclose Diigo sites that I visit and that I would never annotate like, for example, my home banking site.

An interesting project would be to develop an extension that does what the Diigo extension does, but by using only the local storage.

For the moment, beware of browser extensions that "complements" online services. They might call "home" more often than you imagined.