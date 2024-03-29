---
layout: post
title: An Overview of Calabash Android
author: Jonas Maturana Larsen
author_img: jonas.jpg
author_email: jonas@lesspainful.com
published: false
---

Hacking ActivityInstrumentationTestCase2
----------------------------------------

When we started working on LessPainful back before [Calabash-Android](http://github.com/calabash/calabash-android) was extracted from our test setup, we really wanted a general version of `ActivityInstrumentationTestCase2` that would let us reuse the same compiled test app against different apps. However after reading some Android source it was clearly not possible since the test case should look something like this

    public class MyTestCase extends ActivityInstrumentationTestCase2 {
      public static String pkg = "com.ex;
      public static Class clazz = MyActivity.class;
      
      public MyTestCase() {
          super(pkg, clazz);
      }
    }

When an instrumentation either from Eclipse or the command line an instance of `android.test.InstrumentationTestRunner` is created. The `InstrumentationTestRunner` will pass up the extras you provided and an instantiate an object for each of the names in the comma-separated list.
Eg. running `adb shell am instrument -e class com.ex.MyActivity,com.ex.MyActivity2 -w com.ex.test/android.test.InstrumentationTestRunner` result in an instance of `MyActivity` and `MyActivity2`being added to a new `InstrumentationTestRunner`. Later on the runner will actually run the test cases in order.

So, to summarize our problem: At construction test we need a package name and a class that we cannot provide with extras. Other options like using the network or reading from storage are not available at the time the tests are constructed either.

That's what we knew when we gave up.

The hack
--------

Sometime later the discussion came up again because we really wanted to reduce turn around for our users.

Once again we would be reading a lot of code from `android.test` but we gave up again.

Later that day a crazy idea started to take form. And this is the result:

    adb shell am instrument 
    -e class a.c,a.o,a.m,a.dot,a.e,a.e,a.dot,a.dot,a.mm,a.y,a.aa,a.c,a.t,a.i,a.v,a.i,a.t,a.y,com.ex.MyActivity 
    -w com.lesspainful.test/android.test.InstrumentationTestRunner


Let me explain a bit. We created `a.Speller` only contains a static `StringBuffer` and a series of classes like `a.a`, `a.cc`, `a.n7`, `a.dot` that would respectively add lowercase a, capital C, the number seven and a dot to the speller.
Now, when the `InstrumentationTestRunner` get around to instan