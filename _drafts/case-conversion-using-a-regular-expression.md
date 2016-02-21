---
layout: post
title:  "Converting underscore to camelCase using a regular expression"
date:   2015-12-20 23:05:17
categories: php guzzle psr7
---

Recently I needed to convert files using underscore naming strategy for variables (e.g. `$first_name`) and
table names to camelCase (e.g. `$firstName`). Turned out that there's a nice way doing this using a regular
expressing.

<!-- more -->

As a PhpStorm user you can simply use the Search&Replace feature with the *regular expression* mode enabled, but
every other advanced IDE or editor should support this. 

The search term depends a little bit from the surroundings, i.e. if need to be more or less restrictive to avoid
side effects. Let's assume a simple file that only contains underscores in variable names and we want to convert
them to camelCase:

{% highlight php %}
$first_name = 'Dexter';
$last_name = 'Morgan';
{% endhighlight %}

In this case it's enough to use a broad regular expression like  `_(.)` to match everything. Now the trick ist
convert the case when replacing with the first

Blog Post: Regexp replace from underscored to camel case:  => \U$1


{% highlight php %}
...
{% endhighlight %}


