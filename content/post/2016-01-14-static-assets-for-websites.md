+++
Description = ""
Tags = []
date = "2016-01-14T14:22:00+09:00"
title = "Static assets for websites"

+++

Count me in on the developers who believe that [GNU make](http://www.gnu.org/software/make/manual/make.html) is the best tool for assembling static assets.

### The general problem

We need to maintain a set of files B that is derived from another set of files A through some known (and possibly complicated) transformation.  We edit the files in set A but not in set B.  We would like a simple way to (1) create B from A, and (2) update B when A changes, only recreating the parts that are necessary.

### The more specific problem

B is the set of static assets for a web service, and A is the set of source files used to make them.  Only A will be checked into source control, and only B will be uploaded to the web server.

There are different kinds of assets in A that need to be treated differently.

**Javascript**

* My Javascript source files are formatted nicely and full of meaningful, well-thought-out comments.  I would like the js files sent with the web pages to be devoid of comments and mashed together so as to be almost unreadable.  This can be accomplished by piping the files through [JSMin](http://www.crockford.com/javascript/jsmin.html) on the way from A to B.

*  My Javascript source files are modular, and one page may need several files.  These are best combined into one file for faster loading.  Also, any source file could be included in several combination files.  I would like the ability to have each js file in B created from an arbitrary combination of source files from A.

**CSS** 

* All my css is written as scss and needs to be processed with an scss compiler such as  [Sass](http://sass-lang.com/).  Scss files may import other sccs files, a fact we need to be aware of when detecting changes.

Other assets such as images and precompiled libraries can be copied from A to B without modification.

### What to do

The first thing is to define a directory structure.

For set A we'll make a subdirectory `src` in project root with four subdirectories: `js` for Javascript sources, `css` for scss sources, `i` for image files, and `lib` for precompiled libraries.

For set B we'll make a subdirectory `pub` in project root.  Compiled js and css files will go directly in `pub`, and the two subdirectories `i` and `lib` will mirror `src/i` and `src/lib`.

    .
    ├── src
    │   ├── js
    │   ├── css
    │   ├── i
    │   └── lib
    └── pub
        ├── i
        └── lib

Next we need to make a list of the js and css files we would like generated and placed into `pub`.  We'll do that by defining variables `JSFILES` and `CSSFILES`, e.g.:

    JSFILES := main.js eggs.js pancake.js
    CSSFILES := blueberry.css yogurt.css

After that, we need to define the dependencies for each of these files, e.g.:

    pub/main.js: src/js/main.js
    pub/eggs.js: src/js/eggs.js src/js/milk.js
    pub/pancake.js: src/js/milk.js src/js/flour.js src/js/eggs.js
    
    pub/blueberry.css: src/css/blueberry.scss src/css/fruit.scss
    pub/yogurt.css: src/css/yogurt.scss

To simplify things, we'll define the default dependeny to be one source file of the same name, so we can omit dependency definitions for `main.js` and `yogurt.css`.  We'll also define `JS := src/js`, `CSS := src/css` and `PUB := pub`.

    $(PUB)/eggs.js: $(JS)/eggs.js $(JS)/milk.js
    $(PUB)/pancake.js: $(JS)/milk.js $(JS)/flour.js $(JS)/eggs.js
    $(PUB)/blueberry.css: $(CSS)/blueberry.scss $(CSS)/fruit.scss

Finally, we need to make a list of directories to be copied directly from `src` to `pub`.

    COPYDIRS := lib i

This is now enough information for us to build a simple makefile, giving us (at least) the following commands:

* `make` does a clean build, deleting `pub` if it exists and building everything from src.
* `make build` does an incremental build of js and css files, updating only those files whose source has changed.
* `make copy` syncs the directories `i` and `lib` from `src` to `pub`.
* `make watch` runs until you kill it, watching for changes in `src` and updating `pub` as necessary.

Note that `make` is short for `make all`, which does `make clean` + `make copy` + `make build`.

<script src="https://gist.github.com/ders/627147bf67544c96f8be.js"></script>

### How it works

The meat of this makefile is in the pattern rules (lines 43-55).  Quick cheat sheet:  `$@` = target, `$^` = all dependencies, `$<` = the first dependency.  [Details are here.](http://www.gnu.org/software/make/manual/make.html#Automatic-Variables)

The first rule takes care of `main.js` and `eggs.js`.

The second rule takes care of `pancake.js`.  Note that `pancake.js` doesn't match the first rule because there is no source file called pancake.

The third rule takes care of `blueberry.css` and `yogurt.css`.  Note that on line 55 `fruit.scss` is **not** supplied as an argument to sass.  It's only listed as a dependency because `blueberry.scss` contains an `@import "sass";` directive.

Finally, lines 32-36 take care of syncing directories `i` and `lib`.

In the end, our filesystem looks like this:

    .
    ├── src
    │   ├── js
    │   │   ├── eggs.js
    │   │   ├── flour.js
    │   │   ├── main.js
    │   │   └── milk.js
    │   ├── css
    │   │   ├── blueberry.scss
    │   │   ├── fruit.scss
    │   │   └── yogurt.scss
    │   ├── i
    │   │   ├── hanjan.jpg
    │   │   └── ikant.png
    │   └── lib
    │       └── MooTools-Core-1.5.2-compressed.js
    └── pub
    │   ├── i
    │   │   ├── hanjan.jpg
    │   │   └── ikant.png
    │   ├── lib
    │   │   └── MooTools-Core-1.5.2-compressed.js
    │   ├── blueberry.css
    │   ├── eggs.js
    │   ├── main.js
    │   ├── pancake.js
    │   └── yogurt.css
    └── Makefile

### Dependencies

This makefile requires `jsmin`, `sass` and `watchman-make` to be available at the command line.

Jsmin and [Watchman](https://facebook.github.io/watchman/docs/install.html) (which includes watchman-make) are available on OS X via Homebrew.  Sass is not (yet), but it can be installed as a system-wide ruby gem.  I'm not a fan of requiring rubygems for my decidedly anti-rails build system, but since Sass runs nicely from the command line I'll turn a blind eye for now.

Jsmin is also [available via npm](https://libraries.io/npm/jsmin).

### Other features I'd like to include

Would be nice to automatically detect @import statements in scss source files and generate dependency lists based on that.  I'm aware that the Sass package has it's own watcher that handles dependencies, but using that would mean bypassing a significant part of the makefile, thereby making a mess.

It would be pretty simple to add a `make deploy` command to rsync the server.  I'll probably do that later.

### A feature I excluded on purpose

Many web frameworks automatically append timestamps or version numbers to static assets in order to defeat browser caching.  This adds a whole lot of complexity for a pretty minor benefit.  Once a site is in production, I expect updates to be few and far between, and I'm happy to manually add a version number to a target filename as necessary.

### Credits

This Makefile was heavily influenced by and owes thanks to [this blog post](http://west.io/post/2015/04/11-frontend-builds-with-makefiles/).  Thank you!
