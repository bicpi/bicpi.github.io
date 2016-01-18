---
layout: post
title:  "Using Doctrine Naming Strategies in Symfony"
date:   2015-12-20 23:05:17
categories: php doctrine database symfony
---

By default, entity cl

<!-- more -->


{% highlight php %}
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="person")
 */
class Person
{
    /**
     * @ORM\Column(name="first_name")
     */
    private $firstName;
    
    /**
     * @ORM\Column(name="last_name")
     */
    private $lastName;
    
    /**
     * @ORM\Column(name="age", type="integer")
     */
    private $age;
    
    // ...
}
{% endhighlight %}


{% highlight yaml %}
doctrine:
    dbal:
        # ...
    orm:
        # ...
        naming_strategy: doctrine.orm.naming_strategy.underscore
        # ...
{% endhighlight %}

{% highlight php %}
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 */
class Person
{
    /**
     * @ORM\Column
     */
    private $firstName;
    
    /**
     * @ORM\Column
     */
    private $lastName;
    
    /**
     * @ORM\Column(type="integer")
     */
    private $age;
    
    // ...
}
{% endhighlight %}
