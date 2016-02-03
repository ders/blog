+++
Description = ""
Tags = []
date = "2016-02-03T17:51:00+09:00"
title = "Using Grunt to Manage Static Assets"

+++

I [previously posted](/post/2016-01-14-static-assets-for-websites/) about using GNU Make to manage front-end assets for a website.  A colleague suggested that I should check out [Grunt](http://gruntjs.com/getting-started) as it does everything I need to do and more.  So here it is.

I have the same goals as I did last week:

* concatenate an arbitrary combination of js files, minifying them in the process
* preprocess css with sass
* copy directories i and lib untouched
* run a watch process to update files as they're changed

## Installing grunt

Grunt is part of the [node.js](https://nodejs.org/) ecosystem, and as such is available via [the node package manager (npm)](https://www.npmjs.com/).  Npm is available on OS X via Homebrew.

### Basic npm concepts

There are a few things that we need to understand about npm.  The biggest headache was recognizing the difference between local and global installs and knowing when to use which.

* Npm installs packages into a project (unless the `-g` global option is specified, more on that later) and needs to be run in project root.  Packages then go into a subdirectory called `node_packages`.
* If you're in some other directory when running npm, the packages will go into a `node_packages` subdirectory there and confuse you.
* Npm expects to see a file called `package.json` in the project root directory and complains if it's not there.
`package.json` includes a list of packages that the project depends on, and the default `npm install` without any parameters installs those packages.
* When installing a package explicitly, there is in an option to add an entry to `package.json` so that someone else will be able to use `npm install` and get everything.  Note that this is an option and not the default behavior.

### Creating the package.json file

According to [the documentation](http://gruntjs.com/getting-started#package.json),
the command to use is `npm init`, and it must be run in project root.  Running it starts a dialog on the terminal, asking some mostly irrelevant questions: name (defaults to the name of the project directory), version (defaults to 1.0.0), description, entry point (defaults to index.js), test command, git repository, keywords, author, and license (defaults to ISC).  These questions can be suppressed by using `npm init --yes`, which defaults everything.

Unfortunately, npm will complain if it doesn't see a description, a repository field and a license field.  The defaults only cover the license field, leaving the description blank and the repository field missing altogether.

The minimum `package.json` has [just a name and a version](https://docs.npmjs.com/getting-started/using-a-package.json#requirements).
But since I'm [a stickler for getting rid of warnings](https://www.bignerdranch.com/blog/a-bit-on-warnings/), I'm going to have to create my own `package.json` that includes name, version, description, repository and license.  None of this information is relevant; its only purpose is to make the warnings go away.

    {
      "name": "taco",
      "version": "1.0.0",
      "description": "xyz",
      "repository": {
        "type": "git",
        "url": "xyz"
      },
      "license": "ISC"
    }

Unfortunately there's one warning I can't get rid of.  At the time of this writing, `npm install grunt` produces this:

    npm WARN deprecated lodash@0.9.2: lodash@<2.0.0 is no longer maintained. Upgrade to lodash@^3.0.0

According to [the changelog for lodash](https://github.com/lodash/lodash/wiki/Changelog#v092),
version 0.9.2 was released in 2012, and the current version is 4.0.0.  Even the "upgrade to" version of 3.0.0 is a year old already.  This is a red flag; how and why are these dependencies not getting maintained?  That said, it appears that [an update is on the way](https://github.com/gruntjs/grunt/issues/1419).  Will have to ignore this warning for now.

### Grunt plugins

Grunt itself is just the overlord; to do any real work we're going to need some plugins.  After a lot of googling, I've come up with this list:

* To minify and combine javascript files, we can use `grunt-contrib-uglify`.
* To compile scss into css, we can use `grunt-contrib-sass`.
* To copy directories, we can use `grunt-contrib-copy`.
* To delete old files, we can use `grunt-contrib-clean`.
* To watch for changes and recompile, we can use `grunt-contrib-watch`.

All of these are [marked as officially maintained](http://gruntjs.com/plugins), giving us the warm, fuzzy feeling that everything is going to work.

We can now install grunt and the plugins.

    npm install grunt grunt-contrib-uglify grunt-contrib-sass grunt-contrib-copy grunt-contrib-clean grunt-contrib-watch --save-dev

### Grunt command line

There is one more install required if we are to be able to run grunt from the command line.  The package is `grunt-cli`, and needs to be installed globally so that the grunt executable goes into /usr/local/bin and is available in the system path.

npm install grunt-cli -g

It's possible to install `grunt-cli` in the project directory, but then the executable will be in node_modules/.bin instead of /usr/local/bin, and that makes more headaches for us

One gotcha is that the global grunt-cli requires a local grunt or it will fail.  Grunt-cli is a wrapper to find the locally installed grunt to whatever project you're in.  The global grunt-cli will not find a global grunt.

### Summary of grunt installation

* Install npm (e.g. `brew install npm`).
* Create the package.json file shown above.
* `npm install grunt grunt-contrib-uglify grunt-contrib-sass grunt-contrib-copy grunt-contrib-clean grunt-contrib-* watch --save-dev`
* `npm install grunt-cli -g`

`package.json` should go into source control, and `node_modules` should be excluded from source control with the appropriate entry in `.gitignore`.

Once we have `package.json` as updated by the npm install --save-dev command, steps 2 and 3 can be replaced by a simple `npm install`.  We still need to keep step 4; global packages can't go into `package.json` (npm will ignore `--save-dev` when `-g` is specified).

### Optionally installing grunt-cli locally

Installing `grunt-cli` locally instead of globally will allow it to be included in `package.json`, but it has the side effect of not having the grunt executable in the path.  A possible workaround to this side effect is to add a script section to `package.json` with all the grunts you want to do.

    "scripts": { "watch": "grunt watch" }

Then you can type `npm run watch` instead of `grunt watch`.  This may or may not be worth the trouble.


## Writing a gruntfile

### Basic gruntfile concept

The gruntfile is a bit of javascript initialization that gets run whenever grunt is invoked.  The gruntfile needs to define an initialization function and assign that to the global `module.exports`.  Within the initialization function, we'll need to list the modules we need (grunt-contrib-uglify, etc.), specify some configuration for each module, define the default task, and optionally define additional tasks.

Each plugin defines a task of the same name as the plugin (e.g. grunt-contrib-uglify defines an "uglify" task, under which any number of subtasks may be defined).

The gruntfile is named `Gruntfile.js` and resides in project root.  The basic gruntfile structure is:

    module.exports = function(grunt) {
      grunt.initConfig({
        pluginname: { ... }  // one of these for each plugin
      };
      grunt.loadNpmTasks( ... );  // one of these for each plugin
      grunt.registerTask('default', ... );  // define the default behavior of `grunt` with no parameters
      grunt.registerTask( ... );  // optional additional tasks
    }

Each plugin defines a task of the same name as the plugin (e.g. grunt-contrib-uglify defines an "uglify" task, under which any number of subtasks may be defined).  Defining additional tasks is useful for combining tasks into a single command.

A thorough read of [the docs](http://gruntjs.com/getting-started) along with [some examples](https://www.google.co.kr/search?q=gruntfile+examples) gives us enough information to build a single gruntfile, giving us the following commands:

* `grunt` does a clean build, deleting `pub` if it exists and building everything from `src`.
* `grunt build` does an incremental build of js and css files, updating only those files whose source has changed.
* `grunt copy` syncs the directories `i` and `lib` from `src` to `pub`.
* `grunt watch` runs until you kill it, watching for changes in `src` and updating `pub` as necessary.

Note that `grunt` is short for `grunt all`, which does `grunt clean` + `grunt copy` + `grunt build`.

<script src="https://gist.github.com/ders/3ca946b14641e5efe783.js"></script>

### Observations

* Overall, the quality of documentation is poor.  I had to resort to copying examples and then modifying them by trial and error until I got the results I wanted.  There are many alternate syntaxes, causing further confusion.
* Could not find a way to do incremental updates with uglify.  The entire js collection is rebuilt whenever any js source file changes.
* The sass plugin depends on having command-line sass installed as a ruby gem, a dependency that I grudgingly accepted when writing the previous makefile and was hoping to avoid.
* Dependencies from `@import` statements in scss source files are handled nicely; the dependencies are honored when doing an incremental build and don't need to be included in the gruntfile.  This is nice.
* The `grunt-contrib-copy` plugin doesn't know how to sync. The `i` and `lib` directories are copied in their entireties every time there's a change.  There is [another plugin](https://github.com/tomusdrw/grunt-sync) which claims to know how to sync, but I haven't tested it.

### Conclusion

This was a whole lot of trouble to set up a relatively simple build system. Grunt is a powerful tool, and I can see the value of using it when you're already in a node-based project, but it to use it as an isolated build tool is not worth the effort.

The only thing we gained with Grunt is the ability to auto-detect imports in .scss files and do incremental updates accordingly.  At the same time we lost the ability to incremental updates of the Javascript files, at least with the standard plugin.

I was also hoping to avoid the ruby sass dependency by using the plugin, but no luck there since the plugin is just a wrapper for the command line sass.
