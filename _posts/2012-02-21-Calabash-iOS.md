---
layout: post
title: An Overview of Calabash iOS
author: Karl Krukow
author_img: karl.jpg
---

An Overview of Calabash iOS
-------------------------
This post describes [Calabash iOS](http://github.com/calabash/calabash-ios) in some technical detail. It will give you an architectural overview and compare Calabash iOS with some other functional testing alternatives (hence, it is fairly advanced and assumes that you are familiar with [Cucumber](http://cukes.info) and iOS development).

- For an introduction to Calabash and LessPainful (and some info on Cucumber), please read [Calabash: Functional Testing for Mobile Apps](...).
- Instructions for how to use Calabash in your iOS project can be found in the [Github Readme](http://github.com/calabash/calabash-ios).
- For a tutorial on how to use Calabash iOS once setup, please see [Calabash iOS tutorial](http://github.com/calabash/calabash-ios/wiki/Tutorial).



Calabash Architecture
---------------------
Calabash tests are usually run with the [Cucumber tool](http://cukes.info). A Calabash test suite consist of three parts:

- Your app, linked with the `calabash.framework`. In XCode, you make a new test target and link with Calabash only in this target. The framework embeds an HTTP server in your app (of course, this is only in builds of your new test target). The Calabash client library (see next bullet) makes HTTP requests to the server when it wants to do something with the current view.

- The Calabash client library and step definitions. Calabash supports two types of step definitions: [built-in](https://github.com/calabash/calabash-ios/wiki/Predefined,-canned-steps) and custom. The built-ins define general and domain-neutral steps that are intended to give you a quick and easy start. They contain steps like: `I should see a "..." button` and `I swipe right`. The custom steps are written by you, and they're specific to your domain or application. Examples could be: `I should be on the Login screen`, `I login as Pete` or `I add a track to my playlist`. To implement a custom step you can use methods defined in the Calabash *client library*, calabash-cucumber,  which is part of [Calabash iOS](http://github.com/calabash/calabash-ios).

- Your feature files. The feature files describe the app use-cases you want to test. They are simultaneously a specification of the behaviour of your app and an executable test suite.


You can visualize this as:

![Calabash-iOS architecture](img/calabash-ios.png)

Calabash iOS supports running on the iOS simulator as well as (non-jailbroken) iPhones, iPads and iPod touches. The calabash server framework is distributed as a  universal framework for your convenience.

The client library, calabash-cucumber, is installed using [RubyGems](http://rubygems.org/) (see detailed instructions under [Installing the client](https://github.com/calabash/calabash-ios)).


### Comparison with other functional testing systems
There are several options out there for functional testing on iOS. Apple provides its [UIAutomation](https://developer.apple.com/library/ios/#documentation/DeveloperTools/Reference/UIAutomationRef/_index.html), there is [Zucchini](http://www.zucchiniframework.org/), [Frank](https://github.com/moredip/Frank) by Pete Hodgson, [KIF](https://github.com/square/KIF) by square, [FoneMonkey](http://www.gorillalogic.com/fonemonkey) by Gorilla Logic, [iCuke](https://github.com/unboxed/icuke) by Rob Holland, and probably a few I've looked at and forgotten again!

All the projects have their advantages and disadvantages, Calabash included. But we think that Calabash hits a sweet spot that none other the other options cover.


### Frank
[Frank](https://github.com/moredip/Frank) is the library closest to Calabash; in fact, when we started experimenting with iOS support at LessPainful we started with Frank. Frank is a very cool project by a great person (that's you Pete), and Calabash is highly inspired by it. But there were a couple of important reasons why we could not use Frank.

First, Frank is licensed using GNU GPLv3 and we wanted to use the less restrictive EPL. Pete actually didn't want to license Frank as GPL, but since Frank is based on UISpec which is GPL'ed, he had to. We did not want the [UISpec](http://code.google.com/p/uispec/) project as a dependency.

Second, when we started to look at Frank, it was a bit of a pain to set up (in the mean time, its actually gotten much better). It required modification of you app source, inclusion of static resources and source files. We wanted calabash to be easy to set up, so we made Calabash a universal framework.

Third, Frank is focused on running in the iOS simulator. We wanted something that ran equally well on simulator, iPhone and iPad. There is no fundamental reason that the Frank architecture shouldn't work on physical devices (indeed Calabash has the same architecture), but many of Frank's step definitions are simulator only.

Finally, we needed support for some advanced features that Frank just didn't have. Frank's touch emulation is derived from [UISpec](http://code.google.com/p/uispec/), which has some disadvantages: it only supports tapping, and actually doesn't seem to work in all cases ([for example on a UISwitch](https://groups.google.com/group/frank-discuss/tree/browse_frm/month/2011-09/cc1fb65c717180c5)). We wanted support for gestures like swipe, pinch-to-zoom, backgrounding and rotation. We wanted support for querying and acting on UIWebViews.

Frank also has the cool Symbiote tool which let's you explore your app from a browser. We don't have Symbiote in Calabash, but instead we have a powerful console based tool for interacting and exploring your application (more on this in a later blog post).

### KIF, iCuke, FoneMonkey
KIF is another cool project. KIF has an Apache 2.0 license which is ok, but two primary things kept us from going down this route. First, KIF is based on the same touch emulation code as UISpec, and has the same limitations (which actually goes back to a seminal CocoaWithLove post from 2008, [Synthesizing touch events on iPhone](http://cocoawithlove.com/2008/10/synthesizing-touch-event-on-iphone.html)). Secondly, KIF tests are written in Objective-C, and we specifically wanted [Cucumber](http://cukes.info).

[iCuke](https://github.com/unboxed/icuke) actually looked very promising two years ago. Unfortunately its author, Rob Holland, has abandoned it, and we had a difficult time proceeding with it.

FoneMonkey, is GPL licensed and doesn't support Cucumber (also, I have a personal issue with recording-based approaches to test code generation).

### UIAutomation and Zucchini
Apple's UIAutomation framework is actually quite advanced in supporting various gestures. It's also simple to set up since it works without modification to the app (runs via instruments). There are a couple of reasons why we decided not to use UIAutomation.

First, UIAutomation is not open source - this means we have to wait for Apple to make bug fixes, and we can't extend it. It is also very poorly documented by Apple, and not that much information available on the net.

Second, tests are written in JavaScript using a horrible API, resulting in tests that are too easy to break when making minor changes to the UI (IMHO). The tool for developing and running tests (UIAutomation Instrument) gives an inefficient workflow, for example, by locking the editing file, missing the ability to interactively explore your app and running only parts of a test. Also, screenshotting only works on device.

Third, UIAutomation is hard to get working in a continuous integration setup (in fact, when we first looked at it, CI was completely impossible). Extracting tests results and screenshots in a CI friendly way is hard.

[Zucchini](http://www.zucchiniframework.org/), is a new project that's very cool. It has some great addons for working with app screenshots (we might 'borrow' those for Calabash :), and it makes the developer experience much better by using CoffeeScript instead of JavaScript and by coming with some convenience tools. However, by compiling to UIAutomation compatible JavaScript and running via UIAutomation, it has some of the same problems as UIAutomation itself (lack of documentation, Apple control, the fragility of tests, etc.). Also, we didn't want Calabash users to be required to learn CoffeeScript.

### What's next?
In upcoming posts we'll describe more implementation details of Calabash iOS. For example, we'll show how to make advanced custom steps using the client library API, we'll look at support for rotation and gestures like swiping, and even how you can record your own advanced gestures. We'll also look at an efficient workflow for writing tests in an interactive and exploratory manner.


### Discuss!
Discuss this on Hackernews!
