---
layout: post
title: A query language for Android views
author: Karl Krukow
author_img: karl.jpg
author_email: karl@lesspainful.com
published: true
---
#A Query Language for Android Views

*Update*: Karl, February 25th, 2013. Updated with newest features in query. This is not a complete specification. For details refer to [the wiki](https://github.com/calabash/calabash-android/wiki/05-Query-Syntax).

Back in September we published a [roadmap for Calabash in the short to medium term](http://blog.lesspainful.com/2012/09/02/Calabash-Roadmap/). One of the key things on the roadmap was implementing a query language for Calabash Android. The query language will bring additional expressive power to Android tests. Test scripts can become much more dynamic, which removes the need for many of the extensions in the form of custom actions that users of Calabash Android are currently making.

Query-language support has now been released in Calabash Android. So please update to the latest version (0.4.2 at the time of this writing).


Let's see some code!
--------------------

We'll use the WordPress app ([details on how to get up and running here](https://github.com/calabash/x-platform-example). To get started download wordpress using the following (which requires sub-version, svn, installed)

    krukow:~/tmp/query$ mkdir -p android-source
    krukow:~/tmp/query$ cd android-source
    krukow:~/tmp/query/android-source$ svn co http://android.svn.wordpress.org/tags/2.2.7/
    krukow:~/tmp/query/android-source$ cd 2.2.7/
    krukow:~/tmp/query/android-source/2.2.7$ ant debug
    Buildfile: /Users/krukow/tmp/query/android-source/2.2.7/build.xml
    …

Now from the directory containing android-source, connect and Android device (or emulator) and do:

    krukow:~/tmp/query$ calabash-android console android-source/2.2.7/bin/Dashboard-debug.apk
    No test server found for this combination of app and calabash version. Recreating test server.
    Done signing the test server. Moved it to test_servers/63a7fdd310c543c9e27773d4207823d2_0.4.2.apk


Now reinstall and launch:


        irb(main):001:0> reinstall_apps
        …
        => nil
        irb(main):002:0> start_test_server_in_background
        nil

This starts the app on the device:

<img src="/img/cal-android-login1.png" alt="Simple sample app main screen with two buttons and two edit text fields">

Time to actually try query. Let's query for all buttons in the current view:

    irb(main):003:0> buttons = query("button")
    [
        [0] {
                        "id" => "button1",
                   "enabled" => true,
        "contentDescription" => nil,
                     "class" => "android.widget.Button",
                      "text" => "Accept",
                      "rect" => {
            "center_y" => 434.5,
            "center_x" => 89.0,
              "height" => 55,
                   "y" => 407,
               "width" => 146,
                   "x" => 16
               },
               "description" => "android.widget.Button@40573488"
        },
        [1] {
                        "id" => "button2",
                   "enabled" => true,
        "contentDescription" => nil,
                     "class" => "android.widget.Button",
                      "text" => "Decline",
                      "rect" => {
            "center_y" => 434.5,
            "center_x" => 233.0,
              "height" => 55,
                   "y" => 407,
               "width" => 146,
                   "x" => 160
               },
               "description" => "android.widget.Button@40558a10"
        }
    ]

As we can see there are two buttons: one with text `"Accept"` and id `"button1"`, and one with text `"Decline"` and id `"button2"`. We can use `index` to select one of those:

    irb(main):004:0> query("button index:0")
    [
        [0] {
                        "id" => "button1",
                   "enabled" => true,
        "contentDescription" => nil,
                     "class" => "android.widget.Button",
                      "text" => "Accept",
                      "rect" => {
            "center_y" => 434.5,
            "center_x" => 89.0,
              "height" => 55,
                   "y" => 407,
               "width" => 146,
                   "x" => 16
            },
               "description" => "android.widget.Button@40573488"
        }
    ]

#Filtering based on properties

Using `index` often leads to brittle tests that may break if the user interface changes. On iOS there is a concept of a "mark" -- a general way of identifying views.

The notion of `mark` also exists in Calabash Android: here it means view selection by one of the properties `id`, `contentDescription` or simply `text`. Here is an example:

    irb(main):005:0> query("button marked:'Accept'")
    [
        [0] {
                "id" => "button1",
                "text" => "Accept",
                ...
             }
    ]

In general you can filter based on any "property" of a view. A property is defined as any piece of data you can access by calling a getter on the view. (Actually, any simple data you can extract by calling a no-arg method is a "property")

For example, let's filter the buttons by matching against the value of their `getText` method instead:

    irb(main):006:0> query("button text:'Accept'")
    #same result

Right now, only integer, string, boolean and null data types are supported. Strings are written in single quotes `'a string'`. More exact details on what happens is described in the Wiki page about [query syntax](https://github.com/calabash/calabash-android/wiki/05-Query-Syntax).


# Filtering based on classes
The first query we saw in this post was `query("button")` - this is actually querying for all _visible_ views which have _a class with "simple name"_: `"button"` (simple name is the last segment of the fully qualified name). The comparison is case insensitive so this will match the class: `android.widget.Button`.

You can also match on a qualified class name, e.g., `query("android.view.View")` will match all views of type `"android.view.View"` (or subtypes thereof).

There is an important distinction between iOS and Android here: on iOS there is a "magic" translation from say `label` to `UILabel` (the class in `UIView`). There is no such translation on Android, only matching based on (sub-) type of a class or simple name for convenience.

Another important distiction is that only fully qualified queries match also sub-types. So if you have a sub-class of `android.widget.Button` say `com.example.AwesomeButton` this will not be found with `query("button")` but it will with `query("android.widget.Button")`. So, at least right now, the "simple-name" notation is restricted.

Also, there is a short-hand notation for "android.view.View" which is simply "*" (e.g., `query("*")`).

# View hierarchy
On iOS you can navigate the view hierarchy, for example `query("tableViewCell index:0 label")` finds all labels inside the first table cell.

Here is an Android example:

    irb(main):010:0> query("* id:'buttonPanel' button text:'Accept'")
    #same result yet again...

This reads as "find a view with id 'buttonPanel', then inside of that find a button with text 'Accept'".

On iOS you can navigate up and down the hierarchy. This is also supported on Android. Here is a powerful example:

    irb(main):011:0> query("* id:'buttonPanel' button text:'Accept' sibling button")
    [
        [0] {
                        "id" => "button2",
                   "enabled" => true,
        "contentDescription" => nil,
                     "class" => "android.widget.Button",
                      "text" => "Decline",
                      "rect" => {
            "center_y" => 274.5,
            "center_x" => 353.0,
              "height" => 55,
                   "y" => 247,
               "width" => 226,
                   "x" => 240
            },
               "description" => "android.widget.Button@40558a10"
        }
    ]

This finds the "sibling" button of "Accept", namely the "Decline" button. You can also navigate in these directions: descendant, child, parent (descendant being the default).

#Property extraction

Just as on iOS, you can extract properties by providing more than one argument to the query function:

    irb(main):020:0> query("button",:text)
    [
      [0] "Accept",
      [1] "Decline"
    ]

Here, the `android.widget.Button` views are found, and text is extracted via a `text()`, `getText()` or `isText()` method.

Several arguments can be supplied, in which case the process continues:

    irb(main):021:0> query("button", :text, :length)
    [
      [0] 6,
      [1] 7
    ]

Do you dare try this: `query("button", {:setText => "No way!"})`?

# Touch
If you can find it with query, you can touch it! Touching (and other gestures) can be done in two ways. You can provide a query to the `touch` function: e.g. `touch("button text:'Accept'")`. Alternatively you can store the result of a past query in a variable: `btn = query("button text:'Accept'")` and then later `touch(btn)` (provided of course the button is still visible at the same location).

# Conclusion and What's next?

As mentioned in the roadmap, we are working at bringing Calabash Android and Calabash iOS closer together. With query-support in the released 0.4.x line, this is already a big step forward in providing a truely cross-platform testing experience for mobile. We've also provided an example of cross-platform testing with Calabash which maximizes reuse of test-code across platforms.

[Cross-platform example](https://github.com/calabash/x-platform-example)

Getting good community feedback on the query language is important to us. So we encourage everyone to try it out and report experiences and bugs to the [Calabash Android Group](http://groups.google.com/forum/?fromgroups#!forum/calabash-android) and to submit improvements as [pull requests](https://github.com/calabash/calabash-android).

The next blog post will shed some light on how Calabash iOS will improve in the coming months. I think we have some very interesting news here that we're looking forward to sharing with you… Stay tuned.
