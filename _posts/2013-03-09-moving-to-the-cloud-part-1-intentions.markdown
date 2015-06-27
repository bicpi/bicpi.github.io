---
layout: post
title:  "Moving to the cloud part 1: Intentions"
date:   2013-03-09 07:27:19
categories: cloud aws
---

Currently, we are moving our first [Symfony2](http://symfony.com/) application to
the [Amazon cloud](http://aws.amazon.com/) (AWS). This series of articles describes
how we modified and moved this application.

The existing application setup is a common one:

* Single production server
* Usual LAMP stack with Ubuntu and local MySQL database
* File based sessions
* User uploads stored in the local filesystem
* Local [Postfix](http://www.postfix.org/) mailserver
* Cron jobs, e.g. for sending email reports
* Deployment happens from a local machine using a [self written deployment script](https://github.com/bicpi/deployer)
* A little bit of monitoring using [Nagios](http://www.nagios.org)
* Database backups using a self written script
* File backups using [duplicity](http://duplicity.nongnu.org)
* DNS management via client's domain registrar
* Git Version Control using [Bitbucket](https://bitbucket.org)

In the past, every programmer at [our office](http://www.onemedia.de) has been more
or less a one-man show, being the master of all the above mentioned processes. But
our applications have grown up:

* Traffic is rising
* Clients demand more reliability
* Application requirements became far more complex

So in the course of time we were getting concerned with our single server setup because
it is not reliable to have a single point of failure and we are not able to scale in
either direction. Furthermore, programmers need to get back to their real work instead
of struggling with backup scripts, updating server software or monitoring the monitoring
process.

## Cloud to the rescue

As we were using Amazons S3 storage since its launch in 2007 and some of us have already
delved into all the Amazon cloud services added in the last years out of curiosity, it was
no big discussion about what could improve our setup:

* [Route 53](http://aws.amazon.com/route53/) for managing DNS records
* [S3](http://aws.amazon.com/s3/) (Simple Storage Service) for storing user uploads
* [SES](http://aws.amazon.com/ses/) (Simple Email Service) to replace our local mailserver
* [RDS](http://aws.amazon.com/rds/) (Relational database service) to replace our local database server
* [EC2](http://aws.amazon.com/ec2/) (Elastic Compute Cloud) to implement an [autoscaling](http://aws.amazon.com/autoscaling/),
 [load balanced](http://aws.amazon.com/elasticloadbalancing/) server farm
* At a later date, an extended setup may use [ElastiCache](http://aws.amazon.com/elasticache/)
 (Amazon's [memcached](http://en.wikipedia.org/wiki/Memcached)) and [CloudFront](http://aws.amazon.com/cloudfront/)
 (Amazon's [CDN](http://en.wikipedia.org/wiki/Content_delivery_network)) for increased performance

Route 53 will save us from tedious email communication and idle times about everything
where DNS is involved. Switching an A record to another IP or setting up a subdomain? Only
a matter of minutes. We will gain full control and flexibility without the risk of being
offline because our DNS contact at the other end of the world is still at sleep.

S3 will act as a kind of shared file server and solve the handling of user uploads between
the different members of the server farm. So all the files are stored independently from
the load balanced, auto scaled web servers and they are accessible through a well-designed API.

SES will replace the need of a mail server. As our application only sends out emails we will
just have to configure a given SMTP endpoint.

RDS will replace the need of a MySQL server. Security patches will be installed automatically,
hot backups are available at any point in time during the last few weeks, database
replication between different availability zones is self-evident. We will just have
to configure new connection settings and move the existing database to the cloud. For all the
rest we do not lift a finger apart from archiving our obsolete backup script.

EC2 is the heart of the new setup up since this is where our application code is installed.
A template server instance with Ubuntu, Apache and PHP will be used to setup a load balanced
server cluster. By configuring traffic and workload triggers, we will be able to scale the
number of server instances up and down automatically. As they are all based on the same
template instance, they all share the same configuration. Application code will have to be
installed on boot time. Deploying new application code will be the trickiest part.

ElastiCache may be used at a later point in time to store the sessions. It is a memcached
compatible key-value store. For now we will move the sessions to the database.

CloudFront may be used in the future to improve performance. Assets (CSS, JavaScript, images
etc.) can be moved to a special S3 bucket and Amazon will replicate them to edge locations all
over the world.

On top of everything we can take advantage of a lot of security features. For instance, RDS
and the EC2 instances can easily be configured to be only accessible from our own network or
the application server cluster respectively.

While thinking about how to move our prototype application to the cloud it turned out that we
do not need to move everything at once. It should be possible to hook in all the services step
by step into the productive system.

In the next parts of this series I will describe step by step how we modified and moved our
first Symfony2 application to the Amazon cloud:

1. [Intentions (this part)]({% post_url 2013-03-09-moving-to-the-cloud-part-1-intentions %})
2. [Enabling database session storage]({% post_url 2013-03-14-moving-to-the-cloud-part-2-enabling-database-session-storage %})
3. [Enabling Route 53]({% post_url 2013-03-18-moving-to-the-cloud-part-3-enabling-route-53 %})
4. [Enabling S3]({% post_url 2013-05-11-moving-to-the-cloud-part-4-enabling-s3 %})
5. [Enabling SES]({% post_url 2013-05-15-moving-to-the-cloud-part-5-enabling-ses %})
6. [Enabling RDS]({% post_url 2013-05-19-moving-to-the-cloud-part-6-enabling-rds %})
7. Enabling EC2 Elastic Beanstalk

While having gained a lot of AWS knowledge by reading all the documentation and implementing
a lot of proof of concepts we still wanted to be on the safe side before switching to production.
This is why we decided to benefit from the AWS experience of [tecRacer](http://www.tecracer.de/)
- our cloud system administrators.
