---
layout: post
title:  "Run phpspec when saving files with gulp and PhpStorm"
date:   2015-08-01 20:10:19
categories:
    - php
    - phpspec
    - Gulp
    - PhpStorm
    - TDD
    - Code Kata
---

In our weekly Code Kata at [work](http://www.sessiondigital.de/), we're exercising TDD using
[phpspec](http://phpspec.net/). TDD means switching between coding and running the tests all the
time, may causing cramps by abuse of `Cmd+Tab`. So why not automate the test runs whenever a source
file changes and show a decent notification about success or failure?

<!--more-->

This article shows two different approaches to automatically run `phpspec` whenever a source or
spec file was changed in your codebase.


# Gulp File Watcher

[Gulp](http://gulpjs.com/) is a [Node.js](https://nodejs.org/) based task runner. It allows you
to create and run lightweight customized tasks on the command line and provides tons of plugins,
e.g. for desktop notifications or running `phpspec` with specific options – exactly what we need
for our purposes. After installing Node.js, Gulp first needs to be installed globally by using
the Node Package Manager `npm`:

{% highlight bash %}
$ npm install -g gulp
{% endhighlight %}

Gulp also needs to be installed locally for each project that makes use of it – just as well as the
desired Gulp plugins. To install local Node packages, just type a `node install` command right in the
project's root directory:

{% highlight bash %}
$ npm install gulp gulp-notify gulp-phpspec --save-dev
{% endhighlight %}

Packages and their dependencies are installed into the `node_modules/` directory by default, which should
be ignored by your version control system. If you have a `package.json` file in place (which in contrast
should be under version control) to describe your Node dependencies, the  `--save-dev` option appends the
installed packages and their respective versions to the development dependencies section:

{% highlight javascript %}
{
  "devDependencies": {
    "gulp": "^3.8.7",
    "gulp-notify": "^1.5.0",
    "gulp-phpspec": "^0.2.5"
  }
}
{% endhighlight %}

With `package.json`, you or your fellow teammates just need to run `npm install` without any argument to
get the exact same setup of Node module on their machine. Think of it as [Composer](https://getcomposer.org/)
for Node.

Gulp tasks are defined in a file called `Gulpfile.js` in the project's root directory. First, we import
the plugins and assign them to variables:

{% highlight javascript %}
var gulp = require('gulp'); // Gulp task runner
var phpspec = require('gulp-phpspec'); // for running phpspec
var notify = require('gulp-notify'); // for desktop notifications
{% endhighlight %}

Then, we create a stub task that will run the phpspec tests later:

{% highlight javascript %}
gulp.task('test', function() {
  console.log('Run phpspec ...')
});
{% endhighlight %}

You can already try this task on the command line, providing the task name `test`
to the `gulp` command:

{% highlight bash %}
$ gulp test
[12:16:25] Using gulpfile ~/test/Gulpfile.js
[12:16:25] Starting 'test'...
Run phpspec ...
[12:16:25] Finished 'test' after 67 μs
{% endhighlight %}

Now it's time to fill the `test` task with real code:

{% highlight javascript %}
gulp.task('test', function() {
  var options = {
    verbose: 'v',
    notify: true,
    clear: true,
    formatter: 'pretty'
  };
  gulp.src('spec/**/*.php')
    .pipe(phpspec('./bin/phpspec run', options))
    .on('error', notify.onError({
      title: 'Crap',
      message: 'Your tests failed!',
      icon: __dirname + '/node_modules/gulp-phpspec/assets/file-watcher/test-fail.jpeg'
    }))
    .pipe(notify({
      title: 'Success',
      message: 'All tests have returned green!',
      icon: __dirname + '/node_modules/gulp-phpspec/assets/file-watcher/test-pass.jpeg'
    }));
});
{% endhighlight %}

In `options`, you can specify whatever command line options you want to run phpspec with,
e.g. for pretty or verbose output or if you wish to clear the console after every test run.
With `gulp.src('spec/**/*.php')`, Gulp finds all the PHP files in the `spec/` directory
recursively (using a *glob* pattern) and then pipes them into the `phpspec` command with
the options from above using `pipe(phpspec('./bin/phpspec run', options))`. You may need
to adjust the path to the `phpspec` executable. Now, when running `gulp test` in your
project's root folder, every phpspec test gets executed. Depending on test success or failure,
the appropriate method from the [`gulp-notify`](https://www.npmjs.com/package/gulp-notify)
plugin gets called and shows a nice desktop notification – depending on your operating system.

To implement a *watch* functionality that is monitoring files for changes and then
automatically runs the `test` task, we implement another task called `watch`, which
makes use of a native Gulp feature to watch for file changes:

{% highlight javascript %}
gulp.task('watch', function() {
  gulp.watch(['spec/**/*.php', 'src/**/*.php'], ['test']);
});
{% endhighlight %}

The first parameter of `gulp.watch()` indicates the files we wish to watch for changes,
i.e. in our case all the PHP source and spec files. The second parameter specifies the
task(s) to run when a file change is detected, i.e. in our case only the `test` task. Now
you may start the file watcher with `gulp watch`. But Gulp also allows you to specify
a `default` task that gets executed whenever you do not provide a task name to the `gulp`
command:

{% highlight javascript %}
gulp.task('default', ['test', 'watch']);
{% endhighlight %}

In the second parameter, instead of a callback method, we can also provide an array of
existing task names that we wish to run. To run the tests once and then start the file
watcher, there's now nothing more to it than just running:

{% highlight javascript %}
$ gulp
{% endhighlight %}

If we put everything together, we end up with a quite straightforward `Gulpfile.js`
solution, allowing us to watch for file changes and execute the `phpspec` tests:

{% highlight javascript %}
// Gulpfile.js
var gulp = require('gulp');
var phpspec = require('gulp-phpspec');
var notify = require('gulp-notify');

gulp.task('test', function() {
  var options = {
    verbose: 'v',
    notify: true,
    clear: true,
    formatter: 'pretty'
  };
  gulp.src('spec/**/*.php')
    .pipe(phpspec('./bin/phpspec run', options))
      .on('error', notify.onError({
        title: 'Crap',
        message: 'Your tests failed!',
        icon: __dirname + '/node_modules/gulp-phpspec/assets/file-watcher/test-fail.jpeg'
      }))
      .pipe(notify({
        title: 'Success',
        message: 'All tests have returned green!',
        icon: __dirname + '/node_modules/gulp-phpspec/assets/file-watcher/test-pass.jpeg'
      }));
});

gulp.task('watch', function() {
  gulp.watch(['spec/**/*.php', 'src/**/*.php'], ['test']);
});

gulp.task('default', ['test', 'watch']);
{% endhighlight %}

Now just open a console window (e.g. integrated in PhpStorm: `Alt+F12`), work on your code and watch the
test runs every time you save a file (e.g. in PhpStorm: `Cmd+S`).

A successful test run in PhpStorm looks like thi:

![Gulp File Watcher for phpspec]({{ site.baseurl }}/assets/file-watcher/gulp-success.jpg){: .img-responsive }

A failed test run in PhpStorm looks like this:

![Gulp File Watcher for phpspec]({{ site.baseurl }}/assets/file-watcher/gulp-failure.jpg){: .img-responsive }


# PhpStorm file watcher

Modern IDEs also provide solutions to create file watchers.

In my IDE of choice – PhpStorm – this feature is simply called *File Watchers*. There's even built-in
support for popular transcompilers like Sass or CoffeeScript that compile to their CSS or JavaScript
counterpart on file save, respectively. For our purposes we can easily add a custom file watcher by
importing a ready made solution from [Github](https://github.com/vivait/phpspec-PhpStorm-file-watcher).
Save the [raw XML file](https://raw.githubusercontent.com/vivait/phpspec-PhpStorm-file-watcher/master/phpspec-2-watcher.xml)
somewhere and import it from  **Preferences > File Watcher** using the **Import** button.

![PhpStorm File Watcher for phpspec]({{ site.baseurl }}/assets/file-watcher/phpstorm-config.jpg){: .img-responsive }

All the options are quite self-explanatory, but you may adopt some little things like always showing the
console, whether it is an erroneous or a successful test run. That way, you may be more confident that the
tests ran smoothly.

A successful test run in PhpStorm looks like this:

![PhpStorm File Watcher for phpspec]({{ site.baseurl }}/assets/file-watcher/phpstorm-success.jpg){: .img-responsive }

A failed test run in PhpStorm looks like this:

![PhpStorm File Watcher for phpspec]({{ site.baseurl }}/assets/file-watcher/phpstorm-failure.jpg){: .img-responsive }

As you may have have noticed, there is no desktop notifications. Implementing desktop notifications is possible,
but requires additional work that I'll leave as an exercise to the interested reader.

# Gulp file watcher vs. PhpStorm file watcher

Although Gulp requires Node.js, I prefer the Gulp solution to the IDE File Watcher because:

* it works in every IDE or even without
* the Gulpfile can be under version control and easily distributed with a project
* it provides desktop notifications out-of-the-box

