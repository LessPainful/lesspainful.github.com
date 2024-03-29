---
layout: post
title: Testing Multiple Android Apps
author: Jonas Maturana Larsen
author_img: jonas.jpg
author_email: jonas@lesspainful.com
---

Testing Multiple Android Apps
=============================
The default setup for Calabash-Android is to test a single Android application at a time.
`calabash-android gen` wil generate hooks that will install, launch and uninstall at appropriate times to support the single app use case. However, sometimes it can be useful to interact with multiple apps at the same time. This blog post will dig into details of calabash-android that you might not have touched before.



*Notice that this post is written based on the current pre release of Calabash Android (0.4.3.pre3)*


Primer: Calabash-Android architecture
---------------------------------------

Calabash-Android has a client-server architecture where client and server communicate via HTTP. The responsibilites are split like this:

* *Client*
  The client sends commands to the test server using `performAction`, `query` a number of other methods.
  Besides the communication, the client is responsible for installing, starting, stopping the test server.

* *Test Server*
  The test server is an instrumentation app that has special permissions to access your app. When a test server is launched it will start an HTTP server on a specified port, start your app and start waiting for requests.

By the nature of the Android instrumentation framework, each test server can only test a single app, so to achive multi-app-testing we need to do something special.

When you test an app using the generated project structure, the test server will be installed and started on the single device or emulator you have connected to your computer. The test server will listen on port 7102 and `localhost:34777` will be setup to forward to `device:7102` over USB. All requests from the client is sent to this default test server.

So to test multiple apps, all we need to do is to start multiple test servers. We then configure the client to send requests to each app in turn. Let's see how we can do that.

Testing Multiple Apps
---------------------

Let's assume that we have two apps called `app1.apk` and `app2.apk` located in the root of our project folder.

Create the test servers like this:

    calabash-android build app1.apk
    calabash-android build app2.apk


If you put the following in your `features/support/app_installation_hooks.rb` and remove `features/support/app_life_cycle_hooks.rb` you will get two test servers started when executing `calabash-android run`: one listening on port 7103 and another listening on port 7104.

    require 'calabash-android/management/app_installation'

    AfterConfiguration do |config|
      path1 = File.expand_path("app1.apk")
      @@app1 = Calabash::Android::Operations::Device.new(
        self, nil, "34801", path1, test_server_path(path1), 7103)

      path2 = File.expand_path("app2.apk")
      @@app2 = Calabash::Android::Operations::Device.new(
        self, nil, "34802", path2, test_server_path(path2), 7104)
    end

    Before do |scenario|
      extend Calabash::Android::Operations
      @@app2.reinstall_apps
      @@app2.start_test_server_in_background

      @@app1.reinstall_apps
      @@app1.start_test_server_in_background
      set_default_device(@@app1)
    end

All requests including those from predifined steps will now be send to app1 running on `localhost:34801`.

To start communicating with app2 simply call `set_default_device(@@app2)`.

