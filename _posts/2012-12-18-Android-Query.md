---
layout: post
title: A query language for Android views
author: Karl Krukow
author_img: karl.jpg
author_email: karl@lesspainful.com
published: true
---
#A Query Language for Android Views

Back in September we published a [roadmap for Calabash in the short to medium term](http://blog.lesspainful.com/2012/09/02/Calabash-Roadmap/). One of the key things on the roadmap was implementing a query language for Calabash Android. The query language will bring additional expressive power to Android tests. Test scripts can become much more dynamic, which removes the need for many of the extensions in the form of custom actions that users of Calabash Android are currently making.

The first implementation of a query language has now landed in [Calabash Android master](https://github.com/calabash/calabash-android/commit/fc4bddcafc69e625745dca265dad6213bb1e4d7f) branch! It is modeled loosely after the default query language on Calabash iOS, but without the iOS specific quirks.


Let's see some code!
--------------------

To use this, you should use the latest pre-release version.

We'll use the [simple sample app](https://github.com/LessPainful/goto_demoapp). Make sure you delete your test_servers folder. Then run a simple feature to ensure everything is installed.

    krukow:~/tmp/android$ rm -fr test_servers/
    krukow:~/tmp/android$ calabash-android run login.apk 
    No test server found for this combination of app and calabash version. 
    Recreating test server.
	…

	
Now let's run the console to try out `query`:

    krukow:~/tmp/android$ calabash-android console login.apk 
	irb(main):001:0> start_test_server_in_background
	=> nil
	
This starts the app on the device:

<img src="/img/cal-android-login1.png" alt="Simple sample app main screen with two buttons and two edit text fields">	
	
Let's query for all buttons in the current view:

	rb(main):003:0> buttons = query("button")
	=> [{"id"=>"login_button", "enabled"=>true, "contentDescription"=>nil,
	 "text"=>"Login", "qualified_class"=>"android.widget.Button", 
	 "frame"=>{"y"=>322, "height"=>73, "width"=>84, "x"=>135}, 
	 "description"=>"android.widget.Button@40583830", "class"=>"Button"}, 
	 {"id"=>"login_button_slow", "enabled"=>true, "contentDescription"=>nil, 
	 "text"=>"Login (slow)", "qualified_class"=>"android.widget.Button", 
	 "frame"=>{"y"=>322, "height"=>73, "width"=>143, "x"=>234}, 
	 "description"=>"android.widget.Button@405842b8", "class"=>"Button"}]
	
As we can see there are two buttons: one with text `"Login"` and id `"login_button"`, and one with text `"Login (slow)"` and id `"login_button_slow"`. We can use `index` to select one of those:

	irb(main):004:0> query("button index:0")
	=> [{"id"=>"login_button", "enabled"=>true, "contentDescription"=>nil, 
	"text"=>"Login", "qualified_class"=>"android.widget.Button", 
	"frame"=>{"y"=>322, "height"=>73, "width"=>84, "x"=>135}, 
	"description"=>"android.widget.Button@40583830", "class"=>"Button"}]
	
And as with iOS, "everything we can query, we can touch":

	irb(main):005:0> touch("button index:0")
	=> {"bonusInformation"=>[], "message"=>"", "success"=>true}
	
#Filtering based on properties
Using `index` often leads to brittle tests that may break if the user interface changes. On iOS there is a concept of a "mark" -- a general way of identifying views. On Android this means we can select a view based on `id`, `contentDescription` or simply `text` attributes. Here is an example:

	irb(main):006:0> query("button marked:'Login'")
	=> [{"id"=>"login_button", "enabled"=>true, "contentDescription"=>nil, 
	"text"=>"Login", "qualified_class"=>"android.widget.Button", 
	"frame"=>{"y"=>322, "height"=>73, "width"=>84, "x"=>135}, 
	"description"=>"android.widget.Button@40583830", "class"=>"Button"}]
	
	irb(main):007:0> query("button marked:'login_button'")
	=> [{"id"=>"login_button", "enabled"=>true, "contentDescription"=>nil, 
	"text"=>"Login", "qualified_class"=>"android.widget.Button", 
	"frame"=>{"y"=>322, "height"=>73, "width"=>84, "x"=>135}, 
	"description"=>"android.widget.Button@40583830", "class"=>"Button"}]
	
In general you can filter based on any "property" of a view. A property is defined as any piece of data you can access by calling a getter on the view. For example, let's filter the buttons by matching against the value of their `getText` method instead:

	irb(main):008:0> query("button text:'Login (slow)'")
	=> [{"id"=>"login_button_slow", "enabled"=>true, "contentDescription"=>nil, 
	"text"=>"Login (slow)", "qualified_class"=>"android.widget.Button", 
	"frame"=>{"y"=>322, "height"=>73, "width"=>143, "x"=>234}, 
	"description"=>"android.widget.Button@405842b8", "class"=>"Button"}]

Right now, integer, string and boolean data types are supported. Strings are written in single quotes `'a string'`.

Note that Calabash will try to match the property (in this case `text`) against a getter-method of the view, i.e., `getText`, `isText`. For example:

	irb(main):009:0> query("button enabled:false")
	=> []

produces no results (`isEnabled()` gives `false` for both buttons), whereas:

	irb(main):010:0> query("button enabled:true")
	=> [{"id"=>"login_button", "enabled"=>true, "contentDescription"=>nil, 
	"text"=>"Login", "qualified_class"=>"android.widget.Button", 
	"frame"=>{"y"=>322, "height"=>73, "width"=>84, "x"=>135}, 
	"description"=>"android.widget.Button@40583830", "class"=>"Button"},
	 {"id"=>"login_button_slow", "enabled"=>true, "contentDescription"=>nil, 
	 "text"=>"Login (slow)", "qualified_class"=>"android.widget.Button", 
	 "frame"=>{"y"=>322, "height"=>73, "width"=>143, "x"=>234}, 
	 "description"=>"android.widget.Button@405842b8", "class"=>"Button"}]

yield two results.
	
#Filtering based on classes
The first query we saw in this post was `query("button")` - this is actually querying for all visible views which have a class with "simple name": `"button"` (simple name is the last segment of the fully qualified name). The comparison is case insensitive so this will match the class: `android.widget.Button`. 

You can also match on a qualified class name, e.g., `query("android.view.View")` will match all views of type `"android.view.View"` (or subtypes thereof). 

There is an important distinction between iOS and Android here: on iOS there is a "magic" translation from say `label` to `UILabel` (the class in `UIView`). There is no such translation on Android, only matching based on (sub-) type of a class or simple name for convenience.  

#View hierarchy
On iOS you can navigate the view hierarchy, for example `query("tableViewCell index:0 label")` finds all labels inside the first table cell. 

Here is an Android example:

	rb(main):021:0> query("linearLayout index:2 editText")
	=> [{"id"=>"username_edittext", "enabled"=>true, 
	"contentDescription"=>"username", "text"=>"", 
	"qualified_class"=>"android.widget.EditText", "frame"=>{"y"=>112, 
	"height"=>73, "width"=>343, "x"=>137}, 
	"description"=>"android.widget.EditText@4057cbc8", "class"=>"EditText"}]
	
	irb(main):022:0> touch("linearLayout index:2 editText")
	=> {"bonusInformation"=>[], "message"=>"", "success"=>true}
	
On iOS you can navigate up and down the hierarchy, but this is not yet supported on Android - you can only navigate "downwards" (in the descendant direction.)

#Property extraction

Just as on iOS, you can extract properties by providing more than one argument to the query function:

	irb(main):024:0> query("textView", :text)
	=> ["SimpleUI", "Hello!", "Username:", "Password:", "Unknown username"]

Here, the `TextView` widgets are found and searched for a `text()`, `getText()` or `isText()` method. The found method is mapped across the query result (`"text"` and `:text` are equivalent). 

Several arguments can be supplied, in which case the process continues:

	irb(main):028:0> query("textView", :text, :length)
	=> [8, 6, 9, 9, 16]
	

#Conclusion and What's next?

As mentioned in the roadmap, we are working at bringing Calabash Android and Calabash iOS closer together. The end goal is to provide a uniform experience when using Calabash, independently of the operating system. Also we will try to support maximizing reuse of test code across platforms. 

We already have a number of improvements to the query language on Android in our minds, However, the next step is to get community feedback on the query language for Android. So we encourage everyone to try it out and report experiences and bugs to the [Calabash Android Group](http://groups.google.com/forum/?fromgroups#!forum/calabash-android) and to submit improvements as [pull requests](https://github.com/calabash/calabash-android).

The next blog post will shed some light on how Calabash iOS will improve in the coming months. I think we have some very interesting news here that we're looking forward to sharing with you… Stay tuned.