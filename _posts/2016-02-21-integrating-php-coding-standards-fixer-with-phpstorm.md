---
layout: post
title: "Integrating PHP Coding Standards Fixer with PhpStorm"
date: 2016-02-21 14:56:17
---

Fixing coding standard issue individually is very tedious, e.g after running a static code analysis tool
or after being declined by a code sniffer pre-commit hook. To ease these kind of tasks you may consider 
using SensioLabs's awesome little CLI tool called [PHP Coding Standards Fixer](http://cs.sensiolabs.org/).

You can simply run the _PHP Coding Standards Fixer_ recursively on a directory or on a single file and it'll 
make your PHP files mostly compliant with 
[PSR-1](http://www.php-fig.org/psr/psr-1/)/[PSR-2](http://www.php-fig.org/psr/psr-2/) or Symfony coding standards,
respectively. This vastly unburdens you from doing mindless and repetitive tasks. In addition, your coding style 
then relies on accepted standards, so there's no pointless dispute about details.

Using [Homebrew](http://brew.sh/), installing the _PHP CS Fixer_ is as simple as:

{% highlight bash %}
$ brew install homebrew/php/php-cs-fixer
{% endhighlight %}

For other platforms and installation options check out the [project page](http://cs.sensiolabs.org/).

To fix code styles recursively in all PHP files of a directory: 

{% highlight bash %}
$ php-cs-fixer fix /path/to/dir
{% endhighlight %}

To fix code styles in a single PHP file:

{% highlight bash %}
$ php-cs-fixer fix /path/to/file
{% endhighlight %}

There are very fine-grained options to define the fixes to apply, but using the default `symfony` level should
usually be a good choice to be as close as possible to the Symfony defaults. This already saves much time when
applied from time to time or before a new commit to your VCS. Of course, you can also do a dry run first and 
display a diff of all the changes to be made. Check out `php-cs-fixer help fix` for all the details.

You can also integrate the tool into PhpStorm so that you can apply style fixes by executing a custom shortcut. 
This is possible by setting up an _External Tool_ definition in PhpStorm: Got to `Preferences > Tools > External
Tools > + (Create Tool)` and fill in the details about how to execute the _PHP CS Fixer_ (for convenience, you 
can simply copy&paste the field values given below the screenshot):

![Setting up PHP CS Fixer as an external tool in PhpStorm]({{ site.baseurl }}/assets/php-cs-fixer-in-phpstorm-external-tools.png){: .img-responsive }

**Name**<br>
`php-cs-fixer`<br>
**Program**<br>
`/usr/local/bin/php-cs-fixer`<br>
**Parameters**<br>
`--level=symfony --verbose --config=sf23 fix "$FileDir$/$FileName$"`<br>
**Working directory**<br>
`$ProjectFileDir$`
 
Then navigate to `Preferences > Keymap`, search for `php-cs-fixer` and assign a custom shortcut, e.g. `Ctrl+Alt+P`.
Now, when you open up a PHP file in your editor and hit your assigned shortcut, the code gets formatted according to
your external tool definition of `php-cs-fixer`.

Another nice feature of the _PHP CS Fixer_ is to add a `.php_cs` file at the project's root to improve the tool's
integration by giving sensible default options. This way it'll be quite easy for example to additionally enforce PHP's
short array syntax or to give a default list of directories to be analysed.

Of course, it's should also possible to integrate the _PHP CS Fixer_ into an automated process.

Fixing code styles might also be possible by using PhpStorm's `Reformat Code` action, and I also use it from time to 
time to clean things up a bit, but from my experience this tool is a bit greedy and tries to fix things you may not 
want to be fixed, like chained calls of a fluent interface. Moreover, sharing code style definitions across teams becomes
much easier when they are not tied to an IDE and can even be stored within the code repository (`.php_cs` file). On the
contrary what's quite useful I think is PhpStorm's `Optimize imports` action which cleans up and orders all the `use`
statements – at least since PhpStorm is smart enough to not remove imports that are only used within annotations :-)

**Project website**<br>
[http://cs.sensiolabs.org/](http://cs.sensiolabs.org/)<br>
**GitHub repository**<br>
[https://github.com/FriendsOfPHP/PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer)
