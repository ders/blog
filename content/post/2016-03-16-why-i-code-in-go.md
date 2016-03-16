+++
Description = ""
Tags = []
date = "2016-03-16T16:47:00+09:00"
title = "Why I code in Go for server applications"

+++

I've written server applications in Ruby, Python, and Go.  With Ruby I've tried out both Sinatra and Rails; with Python I've used Flask and Django; with Go I've used the net/http package.

There are endless arguments for and against using this framework or that language, and there are many valid reasons to like or dislike a set of tools.  I personally like Django a lot.  But Go has two features that beat the competition when it comes to writing web services: static typing and explicit error handling.

In Ruby, we often find ourselves having to check if a value is nil before processing it.  Anything can be nil, and unexpected inputs often create nil values where we least expect.  If we forget to check just one place in the code, sooner or later it shows up as a 500 error and our service is [broken](https://www.youtube.com/watch?v=nZiDS-4Xd2k).

Test suites should cover this, but it's just as easy to miss one edge case in a test suite as it is to miss one in the main code.

In Go, nothing can be nil (unless it's a pointer, but it's easy to know when a pointer might not have been initialized).  In the case of unexpected input, a variable is set to its zero value (e.g. 0, &#39;&#39;, {}), and the fact that there was unexpected input [is conveyed separately](http://dave.cheney.net/2015/01/26/errors-and-exceptions-redux).

In Python, we often find ourselves having to convert types, especially in the case of numerical inputs into string variables.  Using a string where an int is required will raise a TypeError exception, and casting a non-numeric string to int will raise a ValueError exception.  Here too, it's all too easy to miss one try-except block and get a 500 error.

Again, test suites should cover this, but that means a test for every possible branch in the code.  Again, it's just as easy to miss one edge case in a test suite as it is to miss one in the main code.

In Go, compatible types are checked at compile time, thereby eliminating this source of errors.

I choose Go for the simple reason that most 500-inducing code bugs can be either caught at compile time or avoided entirely.  The result is faster and more stable deployments than the alternatives.

OK, I lied.  I choose Go because I like it.  But this is a great way to justify my choice.
