---
layout: post
title:  "Moving to the cloud part 7: Enabling Elastic Beanstalk"
date:   2013-06-01 08:19:56
categories: cloud aws
---

Finally, after preparing our application during the
[last few months bit by bit]({% post_url 2013-03-09-moving-to-the-cloud-part-1-intentions %}),
it was time for the last and most important step: move the application code to an
auto-scaling multi-server environment in the cloud and do a smooth DNS switch for
the productive application. First we thought doing this with EC2 directly, but
after some investigations we found solutions like Scalr, OpsWorks (the former
Scalarium) or Elastic Beanstalk doing the heavy work for us. We decided in favor
for **Elastic Beanstalk** - let's see why and how.

<!--more-->

The first time I played around with [Amazon's EC2](http://aws.amazon.com/ec2/)
service some years ago I was completely excited about all the possibilities:
Setting up server instances on demand by simple mouse clicks, choosing between
different instance types regarding CPU and RAM size for the specific needs of an
application, wiring several instances together with a load balancer in front -
and only paying for what's currently in use. Suddenly, even small companies became
able to provide highly scalable multi-server environments without buying resources
in advance. No financial risk if the application fails. No sysadmin that needs
hours or days to configure fresh root servers. Easily extendable setup. No long
running contracts. No setup fees.

Mostly, if you find a solution for an issue, you'll get new issues when
implementing this solution. Ok, with EC2 you can start whatever server instance
type you need at any time. You can do this by a mouse click in the Amazon console
or by programmatically using the API. You can create a load balancer and add
instances to it. You can add more instances if the load grows, you can terminate
instances as soon as you no longer need them. You can SSH onto any instance of
your cluster and install packages, application code or cronjobs. This is really
great - but you'll want more for a productive application. You'll want the scaling
to be triggered automatically by the current load. You'll want all instances to
be configured the same. You'll want to roll out new code to all instances in one
go. You'll want some cronjobs to be running only on one instance of the cluster
that never gets terminated by autoscaling. You'll want to have health checks and
monitoring.

There are some services on the internet that help you with exactly this kind of
issues. In other words, they help you manage your Amazon resources with an
abstraction layer on top of the raw Amazon webservices. Those kind of abstraction
layers are called **Platform-as-a-Service (PaaS)** - you get all the single services
combined to the whole application platform. For instance, you can create named
server farms in a nice browser GUI, configure the autoscaling triggers, deploy
application code to all instances at once with no downtime etc. Some of them are
third party services like [Rightscale](http://www.rightscale.com/) or
[Scalr](http://www.scalr.com/). You'll need to pay a service fee on top of the
Amazon resources you're managing with the service. But Amazon itself also provides
two free services fo this purpose in the AWS console: [OpsWork](http://aws.amazon.com/opsworks/)
(formerly [Scalarium](http://scalarium.com)) and [Elastic Beanstalk](http://aws.amazon.com/elasticbeanstalk/).
Scalarium was acquired by Amazon in the beginning of 2013 and is the foundation
of OpsWorks. It makes heavy use of the configuration management software
[Chef](http://www.opscode.com/chef/) and can therefore be highly customized. 
Elastic Beanstalk was the first Amazon PaaS and is less complex. It is using
several Amazon webservices like EC2, Elastic Load Balancer (ELB), S3 and ties them
together with a configuration made by the user.

OpsWorks will probably be the tool of choice for our next cloud migration. But
when we started our migration it was freshly released and at first glance Chef
cookbooks are not your best friends. So for this time, we put our trust in the
fully established and well documented Elastic Beanstalk service which is free
of charge.

In Elastic Beanstalk, you can host multiple applications. Each application allows
you to have multiple environments, so you can for instance setup a *Staging*
environment with only a single and cheap EC2 instance and an autoscaling
*Production* environment using a powerful but expensive EC2 instance type.

## Setup an Elastic Beanstalk application

To create your first PaaS application, login to the
[Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk).
At first, adjust the appropriate region for your needs, then click on *Create New
Application*. Enter a name and description for your application:

![EB-1]({{ site.baseurl }}/assets/aws/eb-1.jpg){: .img-responsive }

Then, step-by-step, you will be guided through the process to setup your first
environment within your application. First, select your solution stack and
if you wish an autoscaling environment. Actual triggers for autoscaling – like
CPU or RAM usage – will be setup later.

![EB-2]({{ site.baseurl }}/assets/aws/eb-2.jpg){: .img-responsive }

In the third step, you can upload your application code as a ZIP package or,
in the first instance, opt for a sample application. We'll go with the sample
application and install a custom application later on:

![EB-3]({{ site.baseurl }}/assets/aws/eb-3.jpg){: .img-responsive }

Provide some info for your environment name, an unique `elasticbeanstalk.com` sub
domain, e.g. `foobar.elasticbeanstalk.com`, and a description. Please note that
your real application URL is fully customizable using
[Amazon Route53]({% post_url 2013-03-18-moving-to-the-cloud-part-3-enabling-route-53 %}).

![EB-4]({{ site.baseurl }}/assets/aws/eb-4.jpg){: .img-responsive }

We don't need any additional resources at this point, so let's just skip the
next step. Regarding the RDS instance, it's even more convenient to do the setup
[manually]({% post_url 2013-05-19-moving-to-the-cloud-part-6-enabling-rds %}).

![EB-5]({{ site.baseurl }}/assets/aws/eb-5.jpg){: .img-responsive }

In the configuration details, you can specify the EC2 instance type for your
environment. Please note: check out the [pricing](http://aws.amazon.com/ec2/pricing/)
before selecting an instance type. Powerful instance types can get *real*
expensive, so you should go with a **micro** instance first. Besides, give
an EC2 keypair so you can SSH into the managed EC2 machines and an email
address for status notifications. You can also specify a health check URL
within your application which will is pinged every few minutes to check if
your application is still alive.

![EB-6]({{ site.baseurl }}/assets/aws/eb-6.jpg){: .img-responsive }

Review all information and finally *Create* your platform:

![EB-7]({{ site.baseurl }}/assets/aws/eb-7.jpg){: .img-responsive }

You can follow the launch process of your first application environment by
looking at the events log. This may take some time to complete.

![EB-8]({{ site.baseurl }}/assets/aws/eb-8.jpg){: .img-responsive }

After the creation process has completed you'll see a green checkmark for
the health status ...

![EB-10]({{ site.baseurl }}/assets/aws/eb-10.jpg){: .img-responsive }

... and you're able to access the sample application by the `elasticbeanstalk.com`
sub domain you specified before, e.g. `http://foobar.elasticbeanstalk.com`.

![EB-12]({{ site.baseurl }}/assets/aws/eb-12.jpg){: .img-responsive }

The EC2 console now contains EC2 instances for your new Elastic Beanstalk
environment:

![EB-13]({{ site.baseurl }}/assets/aws/eb-13.jpg){: .img-responsive }

A load balancer has been set up automatically in front of your EC2
instance(s):

![EB-14]({{ site.baseurl }}/assets/aws/eb-14.jpg){: .img-responsive }

And there's also a new security group for your new environment, allowing
access to the web server on port 80 and via SSH on port 22:

![EB-15]({{ site.baseurl }}/assets/aws/eb-15.jpg){: .img-responsive }

Looking at your S3 console, you'll notice a fresh S3 bucket related to your
Elastic Beanstalk environment. This is were new release versions of your
application code get stored, so they can be deployed and also be roll-backed
easily.

![EB-16]({{ site.baseurl }}/assets/aws/eb-16.jpg){: .img-responsive }

To deploy a custom application, go back to the dashboard of your Elastic
Beanstalk environment and upload a ZIP package of your application code by
clicking on *Upload and Deploy*:

![EB-17]({{ site.baseurl }}/assets/aws/eb-17.jpg){: .img-responsive }

Again, you can follow the environment update by looking at the *Recent
Events*:

![EB-18]({{ site.baseurl }}/assets/aws/eb-18.jpg){: .img-responsive }

As soon as the deployment is complete you should be able to access your
application by entering the chosen sub domain in your browser, e.g.
`http://foobar.elasticbeanstalk.com`.

What actually happens during deployment is that a ZIP archive of your
whole application is uploaded to a dedicated S3 bucket first. Then
a so-called *Application Version* is created from your upload and
a deployment to the Elastic Beanstalk gets triggered. No matter how
many servers are currently running behind the load balancer in your
environment, Elastic Beanstalk takes care of the roll-out to all of
them and switching everything live in the end.

## Customizing

If your application has specific server requirements like additional software
packages, a different document root, specific PHP settings or if you want to add
custom files (e.g. containing specific parameters for your database connection),
it is possible to include an `/.ebextensions` directory in your zipped application
code. All files using the extension `*.config` in this directory can be used to specify
custom configuration. For instance, if you need the `lynx` package installed,
an `.htaccess` secured area, a custom `parameters.yml` in a Symfony project, some
custom `php.ini` settings, a cron job to send out what's in the Swiftmailer
file spool and set an appropriate timezone for the EC2 instances, you may use a
configuration file `/.ebextensions/app.config` containing this:

{% highlight bash %}
packages:
  yum:
    lynx: []

files:
  "/tmp/htaccess":
    mode: "644"
    owner: webapp
    group: webapp
    content: |
      AuthName "My secured area"
      AuthType Basic
      AuthUserFile /var/app/current/.htpasswd
      require user admin
  "/tmp/parameters.yml":
    mode: "644"
    owner: webapp
    group: webapp
    content: |
      parameters:
        database_driver: pdo_mysql
        database_host: omdb.vnbnlwvfvi3zc.eu-west-1.rds.amazonaws.com
        database_port: ~
        database_name: app
        database_user: app
        database_password: JKIAJ97WSC3L9GFZS7P2
  "/etc/php.d/app.ini":
    mode: "644"
    content: |
      date.timezone = Europe/Berlin
  "/tmp/app-crons":
    mode: "644"
    content: |
      MAILTO=cron@app.com
      * * * * * webapp /var/app/current/app/console swiftmailer:spool:send --env=stage > /dev/null 2>&1

commands:
  10_servertime:
    command: cp /usr/share/zoneinfo/Europe/Berlin /etc/localtime && echo 'ZONE="Europe/Berlin"' | tee /etc/sysconfig/clock

container_commands:
  10_parameters:
    command: mv /tmp/parameters.yml app/config/
  20_protection:
    command: 'mv /tmp/htaccess .htaccess && htpasswd -cb .htpasswd admin "B0DEK/TK2k_p0FgPxK1" && chmod 444 .htpasswd'
  30_crons:
    command: mv /tmp/app-crons /etc/cron.d/
{% endhighlight %}

There's lot of [documentation](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers.html)
how you can customize to your specific needs and how to execute custom commands during the
deployment process.

## Setup a deployment script

Since all Amazon web services can be managed by powerful APIs, you can
also do a deployment by wiring together some CLI commands in a
shell script and by using the [AWS Command Line Tools](http://aws.amazon.com/cli)
(which give you the `aws` command):

{% highlight bash %}
# Application name on Elastic Beanstalk
APP="foobar"
# Environment name on Elastic Beanstalk
ENV="staging"
# S3 bucket name of Elastic Beanstalk application
BUCKET="elasticbeanstalk-eu-west-1-294798127440"

# Create a unique version string for this release
VERSION="$ENV-v_`date +%Y-%m-%d_%H-%M-%S`"
# Zip the application
zip -r $VERSION.zip /path/to/your/application

# Push zipped application to S3 - this may take a while
aws --profile $APP s3 cp $VERSION.zip s3://$BUCKET/$VERSION.zip
# Create application version
aws --profile $APP elasticbeanstalk create-application-version --application-name $APP --version-label $VERSION --source-bundle S3Bucket=$BUCKET,S3Key=$VERSION.zip
# Trigger environment update
aws --profile $APP elasticbeanstalk update-environment --environment-name $ENV --version-label $VERSION
{% endhighlight %}

The remaining question here is how to prepare the application code that gets zipped
so that it can run properly and secure on the Elastic Beanstalk environment.
You may need to setup a custom process here, e.g. to get rid of unnecessary files
like development front controllers or uncompressed asset files. You can do this by
by extending the shell script or by using build tools like [Ant](http://ant.apache.org/)
or [Phing](https://www.phing.info/). In the end, what you need to build, is a directory
with your application code completely stripped down to the bare minimum of what's really
needed to run the application, maybe compressed assets and likely you need to include
a configuration file appropriate for your target environment (database settings, API keys etc.
for this specific environment). For the latter, you may also the custom configuration file
feature shown above or environment variables. On Elastic Beanstalk, you can set environment
variables easily in the environment configuration and they are available on every EC2 instance
managed by your environment.

One thing to note is that in a PHP application that uses Composer, you do not have to include
your `/vendor` directory. Elastic Beanstalk makes some assumption about your application,
and if there's a `composer.json` in your application root, it automatically does a
`composer install` during deployment. This will save you a lot of time and bandwidth when
transmitting your zipped application code.

## Autoscaling

Finally, to enable autoscaling, go to the *Configuration* page of your Elastic Beanstalk
environment and enter a range of servers you'll want your application to run on. For instance,
you may want of minimum of 2 servers to be on the safe side, and a maximum of 4 to absorb
peak loads. You can even setup a timetable for autoscaling – e.g. to prepare for your TV
commercials – and also spread the EC2 instances across Availability Zones to be independent
from regional outages etc.

![EB-19]({{ site.baseurl }}/assets/aws/eb-19.png){: .img-responsive }

## More

There are a way more features available on Elastic Beanstalk than be covered in this article.
You can do for example far more customizations, there are special options for all the different
platform stacks, zero-downtime deployments, SSL, usage of custom domains, integration of other
AWS services etc. Just check out [the extensive documentation](http://docs.aws.amazon.com/elasticbeanstalk)
for more information.
