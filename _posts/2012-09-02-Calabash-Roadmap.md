---
layout: post
title: Roadmap for Calabash
author: Karl Krukow
author_img: karl.jpg
author_email: karl@lesspainful.com
published: true
---
#Roadmap for Calabash
In this post we'll talk about the history and future of Calabash. A lot has happened since it was first publicly announced about half a year ago in our introductory blog post on [Calabash](http://blog.lesspainful.com/2012/03/07/Calabash/). We'll look at the roadmap for Calabash as we see it, but first a bit of history to explain why things are as they are… 

Background
----------
<img alt="Calabash Logo" style="float:left; margin-left:10px;margin-right:10px;" src="/img/calabash-logo.png">

In January 2012, we decided to change LessPainful's business model. The most significant change was to open source the software that was used to write automated acceptance and functional tests for our clients' mobile apps: [Calabash](http://calaba.sh/). This was a lot of work, and continually is. Thankfully, it is also rewarding to see a community thrive. 

Infact, it turned out to be the best decision we ever made for LessPainful: it has generated a lot of attention in the mobile space (see [links 1-6](#links)) below), it has created an [active](https://groups.google.com/forum/?fromgroups#!forum/calabash-ios) [community](https://groups.google.com/forum/?fromgroups#!forum/calabash-android) around the software, and it has given LessPainful a number of very serious larger enterprise customers wanting commercial support, training, Calabash extensions and of course our primary product: [Shared and Private Mobile Device Clouds](http://www.lesspainful.com).

**Before Calabash** was made open source, it existed first as a proprietary library for Android. It was based on Robotium but gave a higher level API and supported Cucumber. The library was used for some time to test our clients' Android apps on real Android devices at [LessPainful](http://www.lesspainful.com). It was clear that many of our clients wanted both Android and iOS support. So I implemented a proof-of-concept of the automation infrastructure for iOS which we needed at LessPainful.com (install/uninstall, start/stop, clear device, change language etc). 

After doing the POC, we needed a testing library that would let our clients use Cucumber to test also their iOS apps at LessPainful. The iOS library was almost a straight reimplementation of [Pete Hodgson's Frank](http://testingwithfrank.com/), changing only what was necessary to support running on our device clouds at LessPainful (it did also [change some significant parts like touch synthesis](http://blog.lesspainful.com/2012/03/07/Calabash-iOS/)).

**What all this means** is that Calabash started and has evolved as two separate systems. Although they have the same goals (supporting BDD via Cucumber on Android and iOS), the implementations differ as do the interfaces they expose, apart from Cucumber of course. 

Now, part of the value proposition for Calabash is _reusing as much as possible_ of your testing code when building the same app on iOS and Android  (i.e., reusing features and test scripts). Improving this situation will be one of the top priorities for Calabash in the near future.

Bringing Android and iOS Closer
-------------------------------
<img alt="Android and iOS logos" style="float:right" src="/img/logo_ios_android.gif">
One of the most important goals for Calabash is to reuse features, step definitions and support code across iOS and Android. We've already begun moving in this direction. For example, Calabash Android has moved to embedding an HTTP server instead of the custom TCP based protocol used before. Soon it will also move to a JSON-based protocol similar to Calabash iOS (which is actually more or less the [Frankly protocol](http://testingwithfrank.com/frankly.html)). This moves the server parts of Calabash iOS and Android closer, meaning that clients can be made closer. Here is a good example: **query**.

Currently, Calabash iOS has a query language based on UISpec which was inherited from Frank. This query language is immensely useful when writing test scripts - you can query into any object in the view, find its properties, call methods etc. Calabash Android doesn't support the query language - and it shouldn't since UISpec is designed around iOS. This is an example where Calabash iOS and Calabash Android will both move away from their current implementation and towards a common query language that makes sense on both platforms. We don't know exactly what this language will look like yet but UISpec has done well for us so far - so we will design something inspired by that. 

This will lead to a set of unified APIs that can be used in your iOS and Android test scripts. This is one of our top priorities for Calabash in the near future.

Extending Reach
-------------------------
<a href="http://openjdk.java.net/projects/mlvm/"><img alt="Da Vinci Machine Project" title="Da Vinci Machine Project" style="float:right" src="/img/helicopter.png"></a>
There are many programmers and QA staff out there that are already familiar with JVM-based languages and tools. Many shops have invested in the JVM both as a platform for development and testing. 

While we love Ruby and enjoy working with it, we also understand that not everyone has the time or will to learn a new programming language. So while Ruby is a very nice language to work with and has a great ecosystem and community supporting testing, it does restrict the reach of Calabash.


For this reason and because there is a good interest (more than 20 companies have declared a strong interest in JVM support), we will be developing **JVM support for Calabash**. BDD will be supported on the JVM via Cucumber JVM, but Calabash JVM itself will be independent of Cucumber.


As a proof of concept, and declaration of intent, we have implemented a minimal and prototype JVM version of Calabash iOS. It is implemented in Clojure, but exposes an API that can be called from Java and other JVM-based languages. The project can be found [here](https://github.com/calabash/calabash-ios/tree/master/calabash-jvm), and examples [here](https://github.com/krukow/calabash-jvm-example) (Java) and [here](https://github.com/krukow/calabash-jvm-clojure-example) (Clojure).


While already quite functional, this is only a proof-of-concept. JVM support will not be our first improvement to Calabash. It makes much more sense to first find a good unified API as discussed above. Once we are happy with our unified APIs we can create a JVM implementation much faster.

_JVM support is of high priority to us, but we will develop unified APIs first_.

Other improvements
-------------------
Of course we will continually add features and fix bugs on the Calabash versions you already know. Other potential areas of improvement are

* **Device Cloud**. Better and easier support for running on the LessPainful Device Cloud. There are actually some very juicy bits here that I can't reveal yet :) It will actually be released quite soon, and described in a separate blog post.


* **Better hybrid/webview support**. While our web view support is actually already quite good, it is also fairly low level. You can query for visible elements and text (this is actually the same API on Android and iOS). You can touch visible elements and enter text. This gets you very far on web views, but it would be nice to have higher level APIs for dealing with common HTML5 components.

* **Image analysis support**. We are thinking about looking into better support for comparing images. While we prefer scripts that look for text or UI components in views (since they are more robust), we also recognize that there is value in image-based regression analysis. Nicholas Albion has [already done some work on this](https://groups.google.com/d/msg/calabash-ios/D0Nx5BJIP-s/I6HDV6GO7FEJ) based on ImageMagick and Zucchini.  

* &lt;Insert Your suggestion here!&gt; Calabash is [truely open source](https://github.com/calabash/calabash-android/pulls). We will listen to community need, and we will acommodate (provided of course requests are inlined with the direction of Calabash).


Stay tuned for more news soon!


<a name="links"></a> 
Presentation Links
------------------

1. [Sauce Labs Mobile Testing Summit, SF 2012](http://mobiletestsummit.com/speakers)
2. [QCon London, London 2012](www.infoq.com/presentations/Calabash-Functional-Testing) <- that speaker bio is kinda dated :)
3. [CukeUp, London 2012](http://skillsmatter.com/podcast/home/calabash-an-open-source-automated-testing-technology-for-ios-and-android)
4. [Goto Amterdam, Amsterdam 2012](http://gotocon.com/amsterdam-2012/presentation/Introducing%20Calabash)
5. [Code Motion, Rome, 2012](http://www.youtube.com/watch?v=0an8l1RAe0M)
6. [StarEast, Orlando, 2012](http://www.sqe.com/ConferenceArchive/StarEast2012/ConcurrentThursday.html#T23#T23)