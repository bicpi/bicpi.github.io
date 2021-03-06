---
layout: post
title: "Moving to the cloud part 4: Enabling S3"
comments: true
disqus_identifier: moving-to-the-cloud-part-4-enabling-s3
redirect_from: /cloud/aws/s3/2013/05/12/moving-to-the-cloud-part-4-enabling-s3/
---

Moving to the cloud mostly means moving to a scalable multi-server environement with a load balancer in front. The load balancer redirects a user to an available web server instance of the cluster. Imagine a form with an image file upload somewhere in your application allowing a user to publish an avatar on his profile page. Handling the uploaded file the old way would mean to store it on the current web server's file system. But how could this file be accessed by other web servers of the cluster, e.g. to display the avatar in the user's public profile to users that have been redirected to another instance of the cluster? Moreover, what happens if we want to scale down our multi-server environment – meaning that we may need to shut down a web server that stores uploaded images? One possible solution would be to setup an additional file server for this purpose, not being part of the scaling cluster. All webs servers could access uploaded files at the same central location. But there are several drawbacks with this setup: First, it means setting up and maintaining another server with a different configuration. Second, it means a single point of failure: if our single file server fails then the whole application is concerned – and for the sake of simplicity mirroring the file server is not an option… S3 to the rescue!

[Amazon S3](http://aws.amazon.com/s3) (Simple Storage Service) is a cloud storage: it allows you to store files on Amazon servers. Based on one of the most reliable infrastructures on the internet and with features like permissions and REST/SOAP APIs on top it should perfectly suit our needs for a centralized file storage. There are some nice S3 management tools which allow file handling similar to all the good old FTP clients, e.g. [S3 Browser](http://s3browser.com/) or [CloudBerry](http://www.cloudberrylab.com/). But remember the file upload in our application: we will need to handle files programatically. The S3 documentation describes all the low level commands for file operations: how to upload or download files, how to setup permissions or how to list your existing files. If you do not want to reinvent the wheel you should use the appropriate standard development kit (SDK), abstracting away all the tedious work of crafting restful URLs and looking up all the parameters. Amazon provides SDKs for all major languages, e.g. the [AWS SDK for PHP](http://aws.amazon.com/sdkforphp/).

But we didn't wanted to tie our application completely to S3 by spreading SDK code everywhere, so we went one step further by using the fabulous [Gaufrette library](https://github.com/KnpLabs/Gaufrette) and its [Symfony2 bundle](https://github.com/KnpLabs/KnpGaufretteBundle). Gaufrette provides a filesystem abstraction layer allowing us to handle files independently from the underlying storage system. In other words: our application will use Gaufrette commands to save and retrieve files without knowing the underlying filesystem: Local, FTP, S3, database, Dropbox, APC …

Every installation of our application can be configured separately which filesystem to use. This allows us to continue working with the local filesystem in our development, staging or test environment without being charged by Amazon and to still be able to develop offline.

Installing the AWS S3 SDK for PHP is as simply as adding a new line to your `composer.json`:

{% highlight javascript %}
{
    require: {
      "amazonwebservices/aws-sdk-for-php": "1.x"
    }
}
{% endhighlight %}

Then update your vendors:

{% highlight bash %}
$ composer udpate amazonwebservices/aws-sdk-for-php
{% endhighlight %}

Configure the `AmazonS3` class from the AWS SDK as a service. Usually you'll have to pass an options array with the S3 credentials as constructor arguments:

{% highlight xml %}
<!-- ... -->
<service id="my.storage.s3" class="AmazonS3">
  <argument type="collection">
    <argument key="key">%uploads_s3_key%</argument>
    <argument key="secret">%uploads_s3_secret%</argument>
  </argument>
</service>
<!-- ... -->
{% endhighlight %}

Add the credentials to your `parameters.yml`:

{% highlight yaml %}
parameters:
  # ...
  uploads_s3_key: MyS3Key
  uploads_s3_secret: MyS3Secret
{% endhighlight %}

Now you could grab the service from the container and just use it:

{% highlight php startinline %}
$s3 = $this->container->get('my.storage.s3');
$s3->create_object('my-bucket-name', 'my-file');
{% endhighlight %}

As mentioned above we can get rid of this dependency on the S3 service by using Gaufrette. Install the Gaufrette Symfony2 bundle by adding another line to your `composer.json`:

{% highlight javascript %}
{
    require: {
        "knplabs/knp-gaufrette-bundle": "0.1.x"
    }
}
{% endhighlight %}

Update your vendors again:

{% highlight bash %}
$ composer udpate knplabs/knp-gaufrette-bundle
{% endhighlight %}

Register the bundle in the kernel:

{% highlight php startinline %}
// app/AppKernel.php
// ...
public function registerBundles()
{
    $bundles = array(
        // ...
        new Knp\Bundle\GaufretteBundle\KnpGaufretteBundle(),
        // ...
    );

    return $bundles;
}
// ...
{% endhighlight %}

The container configuration of the Gaufrette bundle allows to register different virtual filesystems in a so called filesystem map. Every filesystem requires an adapter for defining and configuring the underlying real storage. In our example we register a filesystem named `uploads` and make its adapter configurable:

{% highlight yaml %}
# app/config/config.yml
# ...
knp_gaufrette:
  adapters:
    # ...
  filesystems:
    uploads:
      adapter: "%uploads_adapter%"
{% endhighlight %}

Usage of a filesystem is easy. Just retrieve the desired filesystem from the `knp_gaufrette.filesystem_map` service and use its self-explanatory methods to handle files:

{% highlight php startinline %}
// Get filesystem map
$filesystemMap = $this->container->get('knp_gaufrette.filesystem_map');

// Get filesystem
$filesystem = $filesystemMap->get('uploads');

// Write file
$filesystem->write($path, $content);

// Read file
$filesystem->read($path);

// Delete file
$filesystem->delete($path);
{% endhighlight %}

But how to define and switch the adapters? This also happens in the container configuration:

{% highlight yaml %}
# app/config/config.yml
# ...
knp_gaufrette:
  adapters:
    uploads_local:
      local:
        directory: "%uploads_local_directory%"
    uploads_s3:
      amazon_s3:
        amazon_s3_id: my.storage.s3 # Service ID for AmazonS3 class, see above
        bucket_name: "%uploads_s3_bucket_name%"
        options:
          region: "%uploads_s3_region%"
  filesystems:
    # ...
{% endhighlight %}

The first adapter `uploads_local` defines a simple local filesystem with only one parameter: the directory where the files should be stored. The second adapter `uploads_s3` defines what we were waiting for: S3 storage. We have to specify the service key of the above configured AmazonS3 service and additionally pass a bucket name and the appropriate [AWS region](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region).

Now we can complete our `parameters.yml`:

{% highlight yaml %}
parameters:
  # ...
  uploads_adapter: uploads_local # uploads_local|uploads_s3
  uploads_local_directory: "%kernel.root_dir%/../web/uploads"
  uploads_s3_key: MyS3Key
  uploads_s3_secret: MyS3Secret
  uploads_s3_region: s3-eu-west-1.amazonaws.com
  uploads_s3_bucket_name: my-bucket-name
{% endhighlight %}

By switching the `uploads_adapter` value between `uploads_local` and `uploads_s3` we can now switch between local file storage and S3 storage.
