---
layout: post
title: "Optimize Debugging in a Symfony application"
comments: true
disqus_identifier: optimize-debugging-in-a-symfony-application
redirect_from: /php/xdebug/debugging/symfony/2015/08/11/optimize-debugging-in-a-symfony-application/
---

When debugging a Symfony application with Xdebug you might not only want to step through your own custom code, but also through the Symfony code itself. As a lot of Symfony's core code is aggregated and optimized into a single file at `app/bootstrap.php.cache` for performance reasons, you will not feel very comfortable reading non-highlighted, non-indented code of classes from multiple namespaces all assembled in the same file. And this is even true for the `dev` environment, where debugging usually happens. But some simple tweaks in the front controller will allow you to step through the original Symfony code if and only if a debug session was initialized.

For this article we'll use Symfony's development front controller located at `web/app_dev.php`. Usually it looks like this:

```php
// ...

$loader = require_once __DIR__.'/../app/bootstrap.php.cache';
Debug::enable();

require_once __DIR__.'/../app/AppKernel.php';

$kernel = new AppKernel('dev', true);
$kernel->loadClassCache();
$request = Request::createFromGlobals();
$response = $kernel->handle($request);
$response->send();
$kernel->terminate($request, $response);
```

The result in a debugging session is similar to the following screenshot when it comes to Symfony code:

![PhpStorm Symfony Debugging with bootstrap.cache]({{ site.baseurl }}/assets/phpstorm-symfony-debugging-with-bootstrap-cache.png){: .img-responsive }

A dedicated [cookbook in the Symfony documentation](http://symfony.com/doc/current/cookbook/debugging.html) already describes how to disable the `bootstrap.php.cache` file and the class cache:

```php
// ...

// $loader = require_once __DIR__.'/../app/bootstrap.php.cache';
$loader = require_once __DIR__.'/../app/autoload.php';
Debug::enable();

require_once __DIR__.'/../app/AppKernel.php';

$kernel = new AppKernel('dev', true);
// $kernel->loadClassCache();
$request = Request::createFromGlobals();
$response = $kernel->handle($request);
$response->send();
$kernel->terminate($request, $response);
```

But in the end it recommends to revert the code changes after each debugging session. In practice, this would mean to toggle the changes all the time. So why not change the code only once and figure out at runtime if the current request is a Xdebug debugging session? Let's introduce a boolean flag called `$enableCaching` to switch between enabled and disabled caches. The following code corresponds to the default:

```php
// ...

$disableCaching = false;
if ($disableCaching) {
    $loader = require_once __DIR__ . '/../vendor/autoload.php';
} else {
    $loader = require_once __DIR__ . '/../app/bootstrap.php.cache';
}
Debug::enable();

require_once __DIR__.'/../app/AppKernel.php';

$kernel = new AppKernel('dev', true);
if (!$disableCaching) {
    $kernel->loadClassCache();
}
$request = Request::createFromGlobals();
$response = $kernel->handle($request);
$response->send();
$kernel->terminate($request, $response);
```

An initialized Xdebug debugging session can be identified by checking for these things:

* `xdebug` extension must be loaded **AND**
*  one of these must be true:
   * GET parameter `XDEBUG_SESSION_START` is set **OR**
   * Cookie `XDEBUG_SESSION` is set **OR**
   * php.ini setting `xdebug.remote_autostart` is enabled

This results in a slightly adapted development front controller:

```php
// ...

$disableCaching = extension_loaded('xdebug') && (
        isset($_REQUEST['XDEBUG_SESSION_START']) ||
        isset($_COOKIE['XDEBUG_SESSION']) ||
        ini_get('xdebug.remote_autostart')
    );
if ($disableCaching) {
    $loader = require_once __DIR__ . '/../vendor/autoload.php';
} else {
    $loader = require_once __DIR__ . '/../app/bootstrap.php.cache';
}
Debug::enable();

require_once __DIR__.'/../app/AppKernel.php';

$kernel = new AppKernel('dev', true);
if (!$disableCaching) {
    $kernel->loadClassCache();
}
$request = Request::createFromGlobals();
$response = $kernel->handle($request);
$response->send();
$kernel->terminate($request, $response);
```

Contrary to the previous screenshot you'll now see a pretty useful result when it comes to Symfony code:

![PhpStorm Symfony Debugging without bootstrap.cache]({{ site.baseurl }}/assets/phpstorm-symfony-debugging-without-bootstrap-cache.png){: .img-responsive }

Happy debugging!

*If you want learn more about initializing Xdebug debugging session, check out my blog post about [Connecting Xdebug to PhpStorm](http://www.sessiondigital.de/blog/connecting-xdebug-to-phpstorm).*
