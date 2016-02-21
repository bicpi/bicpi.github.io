---
layout: post
title:  "Using Doctrine Naming Strategies in Symfony"
date:   2016-02-20 22:05:17
---

Frequently, I'm stumbling upon very verbose Doctrine metadata definitions that translate each and every
table and column definition from _camelCase_ names to names separated by underscores, e.g. *firstName*
to *first_name*. But did you know that can simply get rid of all this noise by implementing a Doctrine
naming strategy?

<!-- more -->

Let's consider such a verbose sample entity definition for a person:

{% highlight yaml %}
Paymill\MyService\Domain\Person:
    type: entity
    table: person
    # ...
    fields:
        firstName:
            column: first_name
            type: string
        lastName:
            column: last_name
            type: string
        createdAt:
            column: created_at
            type: datetime
{% endhighlight%}

Or, using annotations:

{% highlight php %}
<?php

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="person")
 */
class Person
{
    // ...
    
    /**
     * @ORM\Column(name="first_name", type="string")
     */
    private $firstName;
    
    /**
     * @ORM\Column(name="last_name", type="string")
     */
    private $lastName;
    
    /**
     * @ORM\Column(name="created_at", type="datetime")
     */
    private $created_at;
    
    // ...
}    
{% endhighlight %}
            
It seems quite verbose to translate the *camelCase* entity table name and properties names to *under_score* 
table and column names, e.g. `Person` to `person`, `firstName` to `first_name` etc. Furthermore, it is quite
prone to become inconsistent if someone forgets it for a column that's added later. So why not automate the translation?

[Doctrine Naming Strategies](http://doctrine-orm.readthedocs.org/projects/doctrine-orm/en/latest/reference/namingstrategy.html)
to the rescue! Doctrine naming strategies allow to define how entity class names and 
properties are translated to table and column names if they are not provided by the metadata. By default, Doctrine 
does not change anything and in the example above you'd end up with a `Person` table and the column names `firstName`, 
`lastName` etc. But by configuring the *Underscore Naming Strategy* you'll get exactly what you want – without all the 
table and column config noise in your metadata:

{% highlight yaml %}
Paymill\MyService\Domain\Person:
    type: entity
    # ...
    fields:
        firstName:
            type: string
        lastName:
            type: string
        createdAt:
            type: datetime
{% endhighlight %}       

Or, using annotations:

{% highlight php %}
<?php

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 */
class Person
{
    // ...
    
    /**
     * @ORM\Column(type="string")
     */
    private $firstName;
    
    /**
     * @ORM\Column(type="string")
     */
    private $lastName;
    
    /**
     * @ORM\Column(type="datetime")
     */
    private $created_at;
    
    // ...
}    
{% endhighlight %}

How to set this up? If using Symfony, it's as simple as adding a configuration parameter 
`doctrine.orm.entity_managers.default.naming_strategy` to your `config.yml`:

{% highlight yaml %}
doctrine:
    dbal:
        # ...
    orm:
        # ...
        entity_managers:
            default:
                naming_strategy: doctrine.orm.naming_strategy.underscore
{% endhighlight %}
                
If using plain PHP, you can check the 
[Doctrine docs](http://doctrine-orm.readthedocs.org/projects/doctrine-orm/en/latest/reference/namingstrategy.html)
for the few lines you need to add.

By providing a custom implementation of `Doctrine\ORM\Mapping\NamingStrategy`, you can also easily define
your own naming strategy. This comes in handy if for example you need to follow naming guidelines like prefixing
every table name, e.g. by `tbl_`.

By the way, as `string` is always the default data type, so you can even skip this config for simple columns by only 
putting a tilde `~` character:

{% highlight yaml %}
Paymill\MyService\Domain\Person:
    type: entity
    # ...
    fields:
        firstName: ~
        lastName: ~
        createdAt:
            type: datetime
{% endhighlight %}

When using annotations, just leave out the `type` definition:

{% highlight php %}
<?php

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 */
class Person
{
    // ...
    
    /**
     * @ORM\Column(type="string")
     */
    private $firstName;
    
    /**
     * @ORM\Column(type="string")
     */
    private $lastName;
    
    /**
     * @ORM\Column(type="datetime")
     */
    private $created_at;
    
    // ...
}
{% endhighlight %}

This will finally result in the following SQL definition, which is the same as for the verbose metadata definition 
from the beginning:

{% highlight sql %}
CREATE TABLE `person` (
/* ... */
`first_name` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
`last_name` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
`created_at` datetime NOT NULL,
PRIMARY KEY (`person_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
{% endhighlight %}
