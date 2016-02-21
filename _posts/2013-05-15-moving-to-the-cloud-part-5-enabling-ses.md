---
layout: post
title: "Moving to the cloud part 5: Enabling SES"
comments: true
disqus_identifier: moving-to-the-cloud-part-5-enabling-ses
redirect_from: /cloud/aws/ses/email/swiftmailer/2013/05/15/moving-to-the-cloud-part-5-enabling-ses/
---

Sending emails from a Symfony2 application is no challenging task. Just configure the Swiftmailer library with a handful of simple parameters, create a message object, trigger the sending process and you are done. Things change slightly if you are responsible of the mail server at the same time. Setting up and maintaining mail server software may become a challenging task, especially if there are complaints about missing emails, security holes or spam issues. Moreover, sending from a cloud server is not very reliable because of its doubtful IP reputation. Amazon's SES service provides relief.

[Amazon's SES](http://aws.amazon.com/ses) (Simple Email Service) service frees us from setting up and maintaining a mail server by providing an email service with a single SMTP endpoint.

Switching to SES is as simple as updating your Swiftmailer configuration. It should look similar to the following:

{% highlight yaml %}
// app/config/config.yml
# ...
swiftmailer:
  transport: "%mailer_transport%"
  host: "%mailer_host%"
  port: "%mailer_port%"
  encryption: "%mailer_encryption%"
  username: "%mailer_user%"
  password: "%mailer_password%"
{% endhighlight %}

It should be noted that the `encryption` parameter is not part of the default configuration and no encryption is used by default. Since SES requires TLS encryption we need to add this setting.

If you've signed up for AWS you can find your personal SMTP settings in the [SES Management Console] (https://console.aws.amazon.com/ses) and you can complete your `parameters.yml` with it:

{% highlight yaml %}
parameters:
  # ...
  mailer_transport:  smtp
  mailer_host: email-smtp.us-east-1.amazonaws.com # to be found in the SES console
  mailer_user: SES-User # to be found in the SES console
  mailer_password: SES-Password # to be found in the SES console
  mailer_encryption: tls # TLS encryption required
  mailer_port: 25 # 25, 465 or 587
{% endhighlight %}

That's all – all your email will be sent through SES now and you can start thinking about shutting down your mail server.

Remember that Amazon insists on high quality emails to prevent abuse. Otherwise your account may be disabled.

**Sandbox mode**
A fresh SES account runs in a sandbox mode. In the sandbox mode Amazon limits the number of messages you can send per day and only allows verified sender and recipient addresses. You can verify addresses from the SES console.

**Production mode**
Once you want to switch to production you can request production access from within the SES console. Production mode raises the message limit per day and only requires verified sender addresses. You can also verify a whole domain as a sender domain.
