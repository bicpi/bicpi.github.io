---
layout: post
title:  "Moving to the cloud part 2: Enabling database session storage"
date:   2013-03-14 21:13:06
categories: cloud aws database mysql
---

By default, PHP persists every user session to a single file stored in the system's default temporary directory. You can go there, open an arbitrary session file – most likely prefixed by `sess_` – and you will find a serialized array which represents the contents of the global `$_SESSION` array which is available to your scripts. Ok, this works great, so what's the problem with this setup?

Actually, there is nothing wrong with using the file based session storage. But with growing demands some downsides of this approach may attract your attention:

* The system's temporary directory is a shared directory: session files of different applications &nbsp;and temporary files of foreign processes may also use this location. In case of a security issue your user sessions may be compromised. This may be solved by configuring a unique session save path per application and put an `open_basedir` restriction on top to prevent unauthorized access. This applies all the more if your application is installed on a shared server. In contrast, a database can make use of its access management, you will just need to setup an excluvise account for your session table.
* There are no simple means to increase file access performance. In constrast, a database usually knows a lot of concepts to improve performance like indexing and clustering.
* As soon as you want to run your application on multiple servers for reliability and performance reasons you will prefer to store session data in a central location that is common to all webservers. Thus, every webserver shares the same pool of session data and it doesn't matter which webserver of your cluster serves two subsequent requests of the same client.

Implementing a different session save handler in raw PHP is quite well described in the [PHP documentation][1], so we will focus on how to do the Symfony2 configuration for this requirement. Open your `config.yml` and make the sessions save handler service configurable:



    # ...
    framework:
        # ...
        session:
            handler_id: %session_handler_id%
        # ...


For using the default file based session handling you need to assign the `session.handler.native_file` service in your `parameters.yml`:



    parameters:
        # ...
        session_handler_id: session.handler.native_file
        # ...


In order to switch to database based session handling you change this parameter to `session.handler.pdo` and add some further parameters for the database connection:



    parameters:
        # ...
        session_handler_id: session.handler.pdo
        session_pdo_dsn: "mysql:host=localhost;dbname=session"
        session_pdo_user: myapp
        session_pdo_password: *****
        session_pdo_options:
            db_table: myapp
            db_id_col: id
            db_data_col: value
            db_time_col: time
        # ...


Session data will now be stored in the `value` column of the `myapp` table in the `session` MySQL database on localhost. It is identified by the session ID stored in the `id` column and its expiration date can be derived by the garbage collector from the timestamp saved to the `time` column on every access. Using any convenient database management tool, you can watch the session data change while requesting some webpages of your application.

Setting up the `session.handler.pdo` service is as simple as adding two new service definitions to your service container configuration. Symfony2 already provides a full implementation of a PDO session handler which makes use of the native PDO class.



    services:
        pdo:
            class: PDO
            arguments:
                dsn: %session_pdo_dsn%
                user: %session_pdo_user%
                password: %session_pdo_password%

        session.handler.pdo:
            class:     Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler
            arguments: [@pdo, %session_pdo_options%]


Last but not least you need to create the `myapp` table in the session database:



    CREATE TABLE `session`.`myapp` (
        `id` varchar(255) NOT NULL,
        `value` text NOT NULL,
        `time` int(11) NOT NULL,
        PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;


Maybe you are tempted to create an entity class for this and let Doctrine do all the hard stuff. But do you remember that everlasting mantra of separation of concerns? Session handling has nothing to do with your data model, you will never access it from your custom code. Furthermore, it is the choice of every installation environment and its `parameters.yml` which strategy to use for session handling. Switching to file or memcached based session handling is a perfectly valid option. No need to say that an orphaned session data model would lead to some kind of code pollution in this case. It's also best to use a separate database for the session table(s) so you can still use all the destructive Doctrine console commands in your development environment (dropping database or schema) without the need of recreating your session table afterwards.

[1]: http://www.php.net/manual/de/class.sessionhandler.php
