---
layout: post
title: Test engineering for Cross-platform testing
author: Karl Krukow
author_img: karl.jpg
author_email: karl@lesspainful.com
published: true
---

Test engineering for Cross-platform
====================================

One of the promises of Calabash is _cross-platform_ testing. What we mean by this is the ability to maximize code- and specification reuse when developing the same app (or similar apps) for multiple platforms (e.g. iPhone, iPad, Android Phone, Android Tablet, Mobile Web, Desktop Web,...).

However, it may not immediately apparent how to do this. We do not recommend the use of the Calabash pre-defined steps - these are there only for beginners and to get results quickly. For larger, serious test suites, pre-defined steps should be completely avoided.

We've written an [article about cross-platform testing with Calabash](https://github.com/calabash/calabash-ios/blob/0.9.x/calabash-cucumber/doc/x-platform-testing.md) which comes along with the accompanying [sample code](https://github.com/calabash/x-platform-example) based on the open source Wordpress app.

LessPainful also offers a [number of services](https://www.lesspainful.com/pricing) to support the [Calabash](http://calaba.sh) project. One of the most popular services has been on-site training for companies adopting Calabash. In the training class we teach how you use Calaash efficiently as well as cross-platform and test engineering best practices. Contact us as [contact@lesspainful.com](mailto:contact@lesspainful.com) for more information.


Cross-platform testing does not mean 100% code-reuse across platforms. This may seem disappointing at first, but when you consider the fact that well-designed UIs are different for different platforms, even if you're build the same app, then this becomes immediately clear. Instead, what we should hope for is to reuse the parts of the testing code that should be the same: the specifications and step definitions.

For example consider these screens from the wordpress app:


<img src="/img/wordpress_apps.jpeg" style="width:80%; height:80%;">
