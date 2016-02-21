---
layout: post
title: "Moving to the cloud part 3: Enabling Route 53"
date: 2013-03-18 08:33:17
redirect_from: /cloud/aws/route53/dns/2013/03/18/moving-to-the-cloud-part-3-enabling-route-53/
---

This is part 3 of [my series of articles]({% post_url 2013-03-09-moving-to-the-cloud-part-1-intentions %})
about our first application move to the Amazon Cloud. As we are not the owner
of the productive domain of the application, all communication about DNS changes
has always been quite tedious, time consuming and prone to errors in the past –
especially when your contact person lives in another time zone. In preparation
to the final application move to EC2, which involves some DNS changes to switch
to the Amazon load balancer, we wanted to gain some flexibility. Enter Route 53.

[Route 53](http://aws.amazon.com/route53/) allows you to manage all the DNS records
of a given domain – even if you are not the owner of the domain. For instance, you
can route a domain or any subdomain at any time to any server of your choice. This
is quite cool because whenever we'll be ready with the setup of our EC2 server
cluster we ourselves will be able to flip the switch. No need to contact someone,
no need to wait impatiently for the changes to happen. If something goes wrong –
let's do a rollback to the previous setup.

To let Route 53 take over control your domain's DNS we just have to change the
nameserver records to Amazon's nameservers. Go to the [Route 53 console](https://console.aws.amazon.com/route53),
create a new hosted zone, enter the domain name and save. After saving, the console
lists the names of four Amazon nameservers.

![Route53-1]({{ site.baseurl }}/assets/aws/route53-1.png){: .img-responsive }

You, your registrar or your client's registrar will have to update your domain NS
records with these nameservers. In our case we had to ask our client's registrar
to do this. But wait! As we are dealing with a productive application we should put
the existing DNS configuration in place before requesting the nameserver switch –
otherwise our application would not be accessible any more. Having our application
running at `myapp.onemedia.de` and the code be hosted at `192.51.100.234` we would
just have to create an appropriate A record:

![Route53-2]({{ site.baseurl }}/assets/aws/route53-2.png){: .img-responsive }

![Route53-3]({{ site.baseurl }}/assets/aws/route53-3.png){: .img-responsive }

Now we have some kind of shadow configuration, i.e. we've created a clone of the
existing live DNS configuration which is not yet in use. So we are prepared to
switch the nameservers to Amazon and everything will just work as before – but
then we are in control of the domain's DNS records.

Unfortunately something unexpected happend when we finally requested the nameserver
switch at our client's contact person. Along with replacing the nameservers the
registrar also flushed the existing configuration – more precisely the A records
we have also setup in Route 53. During the next hours we had lots of complaints
about the application being offline and this was quite a painful situation. Moreover,
reachability seemed to be quite arbitrary: some users had no issues while others
were able to reach some of the subdomains only or even nothing at all. After 
thinking a while about what had happend we came to the conclusion that all 
the networks that requested the domain the day before the nameserver switch were 
not able to reach our application any more. The reason was the DNS cache of the
network routers or the computers respectively. We could only wait for the time to
live (TTL) of the nameserver records to expire and ask for understanding. Nonetheless,
we learned our lesson for the next time and there are two possibilities what needs
to be communicated to the registrar:

* Replace the nameserver records but leave the current configuration in place for
 at least one day
* Reduce the TTL for the nameserver records one or two days in advance – then
 change the nameserver records and flush the configuration

After about 18 hours everything was back to normal and we were happy to announce
our first Amazon webservice to be live: Route 53
