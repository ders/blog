+++
date = "2013-03-15T16:59:00+09:00"
title = "Good Tools"

+++

Here are some some tools that I like.

Rerun
-----

> Rerun launches your program, then watches the filesystem. If a relevant file changes, then it restarts your program.

I had some trouble getting Shotgun configured probably, and I found [Rerun](https://github.com/alexch/rerun) to be a simple alternative.  I like it because it can be used for any process you want, not just restarting your web server.


Gitk
----

[Gitk](http://gitk.sourceforge.net/) is a graphical interface for Git.  It shows your entire development tree graphically, great for those of us less fluent in pushes and pulls.  It does a lot of other things too that I don't know about.

Debian-based systems can enjoy Gitk with a simple `sudo apt-get install gitk`.  Then issue a `gitk --all` to see everything.


Irb
---

[Irb](http://ruby-doc.org/docs/ProgrammingRuby/html/irb.html) is the Ruby console.  I should actually generalize this and include all consoles.  One of the most valuable checks while coding is to paste snippets of code into the console and see that they do what you think they're doing.

Make an `.irbrc` startup file for a richer experience.  Rails users can get a the console with the full Rails evnironment with `rails c`.

Here is [an excellent blog post](http://samuelmullen.com/2012/07/getting-more-out-of-the-rails-console/) on the topic.


Awesome Print
-------------

A great companion to the Ruby console is [Awesome Print](https://github.com/michaeldv/awesome_print), a Ruby gem which lets you pretty up the output of array, hashes, and the like while in the console.

{{< highlight ruby >}}
1.9.3-p392 :001 > {:name => "Hong Gildong", :age => 10, :interests => nil}
 => {:name=>"Hong Gildong", :age=>10, :interests=>nil}
1.9.3-p392 :002 > ap _
{
         :name => "Hong Gildong",
          :age => 10,
    :interests => nil
}
{{< /highlight >}}


Sass
----

> Sass makes css fun again.

[Sass](http://sass-lang.com/) is a nifty program that adds some badly-needed css functionality, such as variables, nesting and inheritance.  But the really nifty thing is how it's accomplished.

You make a `.scss` file and Sass converts it to an equivalent `.css` that any browser, even IE, can understand.  Sass even has a stealth mode, where it will regenerate the css when your scss is updated.

Sass is packaged as a Ruby gem, but you're not limited to using it on Ruby projects, since in the end all you need on your server (obviously) are the css files it generates locally.

It's likely that a lot of Sass's functionality will eventually become standard css.  But for now, we've got Sass.
