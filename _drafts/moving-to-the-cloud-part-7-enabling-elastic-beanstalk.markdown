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

To create your first PaaS application, login to the
[Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk).
At first, adjust the appropriate region for your needs, then click on *Create New
Application*. Enter a name and description for your application:

[![EB-1]({{ site.baseurl }}/assets/aws/eb-1-300x156.jpg)]({{ site.baseurl }}/assets/aws/eb-1.jpg)

Then, step-by-step, you will be guided through the process to setup your first
environment within your application. First, select your solution stack and
if you wish an autoscaling environment. Actual triggers for autoscaling will be
setup later.

[![EB-2]({{ site.baseurl }}/assets/aws/eb-2-300x172.jpg)]({{ site.baseurl }}/assets/aws/eb-2.jpg)

In the third step, you can upload your application code as a ZIP package or, in
the first instance, opt for a sample application. We'll go with the sample
application and install a custom application later on:

[![EB-3]({{ site.baseurl }}/assets/aws/eb-3-300x172.jpg)]({{ site.baseurl }}/assets/aws/eb-3.jpg)

Provide some info for your environment name, an *elasticbeanstalk.com* sub
domain and a description. Please note that your real application URL is fully
customizable using [Amazon Route53]({% post_url 2013-03-18-moving-to-the-cloud-part-3-enabling-route-53 %}).

[![EB-4]({{ site.baseurl }}/assets/aws/eb-4-300x172.jpg)]({{ site.baseurl }}/assets/aws/eb-4.jpg)

We don't need any additional resources at this point, so let's just skip the
next step. And for the RDS instance, it's more convenient to do the setup
[manually]({% post_url 2013-05-19-moving-to-the-cloud-part-6-enabling-rds %}).

[![EB-5]({{ site.baseurl }}/assets/aws/eb-5-300x172.jpg)]({{ site.baseurl }}/assets/aws/eb-5.jpg)

In the configuration details, you can specify the EC2 instance type for your
environment. Please note: check out the [pricing](http://aws.amazon.com/ec2/pricing/)
before selecting an instance type. Powerful instance types can get *real*
expensive. Besides, give an EC2 keypair so you can SSH into the managed EC2
machines and an email address for status notifications. You can also specify
a health check URL which will is pinged every few minutes to check if you're
application is still alive.

[![EB-6]({{ site.baseurl }}/assets/aws/eb-6-300x209.jpg)]({{ site.baseurl }}/assets/aws/eb-6.jpg)

Review all information and finally *Create* your platform:

[![EB-7]({{ site.baseurl }}/assets/aws/eb-7-300x286.jpg)]({{ site.baseurl }}/assets/aws/eb-7.jpg)

You can follow the launch process of your first application environment by looking at the events
log. This may take some time to complete.

[![EB-8]({{ site.baseurl }}/assets/aws/eb-8-300x210.jpg)]({{ site.baseurl }}/assets/aws/eb-8.jpg)

After the creation process has completed you'll see a green checkmark for the
health status ...

[![EB-10]({{ site.baseurl }}/assets/aws/eb-10-300x212.jpg)]({{ site.baseurl }}/assets/aws/eb-10.jpg)

... and you're able to access the sample application by the *elasticbeanstalk.com*
sub domain you specified before.

[![EB-12]({{ site.baseurl }}/assets/aws/eb-12-300x239.jpg)]({{ site.baseurl }}/assets/aws/eb-12.jpg)

The EC2 console now contains instances for your new Elastic Beanstalk environment:

[![EB-13]({{ site.baseurl }}/assets/aws/eb-13-300x207.jpg)]({{ site.baseurl }}/assets/aws/eb-13.jpg)

A load balancer has been set up automatically:

[![EB-14]({{ site.baseurl }}/assets/aws/eb-14-300x207.jpg)]({{ site.baseurl }}/assets/aws/eb-14.jpg)

And there's also a new security group for your new environment:

[![EB-15]({{ site.baseurl }}/assets/aws/eb-15-300x207.jpg)]({{ site.baseurl }}/assets/aws/eb-15.jpg)

Looking at you S3 console, you'll notice a fresh S3 bucket named related to your Elastic
Beanstalk environment.

[![EB-16]({{ site.baseurl }}/assets/aws/eb-16-300x207.jpg)]({{ site.baseurl }}/assets/aws/eb-16.jpg)

To deploy you custom application, go back to the dashboard of your Elastic Beanstalk
environment and upload a ZIP package of your application code by clicking on *Upload
and Deploy*:

[![EB-17]({{ site.baseurl }}/assets/aws/eb-17-300x160.jpg)]({{ site.baseurl }}/assets/aws/eb-17.jpg)

Again, you can follow the environment update by looking at the recent events:

[![EB-18]({{ site.baseurl }}/assets/aws/eb-18-300x153.jpg)]({{ site.baseurl }}/assets/aws/eb-18.jpg)

