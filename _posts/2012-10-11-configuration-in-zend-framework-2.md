---
layout: post
title: "Configuration In Zend Framework 2"
description: ""
category: Devel
tags: [ZF2, PHP]
---
First of all, to be honest - Zend Framework 2 is not ready for use at all. Of course, it brings many interesting concepts, but the problem is that you need to dig them out the hard way with numerous WTFs all the time. The documentation is incomplete and there are no best practices yet. The guys designing ZF2 made it extremely modular with components that are loosely coupled and reusable. Anti-patterns like the singleton pattern has been thrown away and dependency injection "rules them all". The question is - how to assemble a working application? The simple answer is - as you like. As a side effect to the modularity, different pieces of configuration,  autoloading and bootstrapping are scattered all over the directory structure and it's up to you what "strategy" to use. Nevertheless, there are some notions of best practices, which can be found in the documentation or in some tutorials. In this post I'll begin with the configuration.

At first, I was really confused by the different configuration files used in ZF2. We have `config/application.config.php`, which seems to be the main configuration file. Then we have the `config/autoload` directory with the `global.php` and `local.php` files in it. Is this some sort of how to configure the autloading or what? Then we have at least one module and its configuration stored in `module/MyModule/config/module.config.php`. So what exactly goes where? And am I supposed to keep my module configuration strictly in its configuration file. What if I had many modules?

Actually the configuration system makes sense and it's not event too complicated. It's just not so evident. The `config/application.config.php` actually configures how your application is configured - it contains information about what modules to use, where to find them and how to configure them. Modules are designed to be self-contained - everything is there - configuration, code, views, tests etc. You should be able to plug a module to an application and it will "just work". But in reality, you will need to alter the original configuration to suit your current application. But you shouldn't do that directly in the module's configuration. The module configuration itself should contain default values. These values can be then overridden in the global configuration by values which suit your environment.

For example, let's have a module *MyModule*, which uses a database connection. We don't want to put real configuration data in the module configuration. So in `module/MyModule/config/module.config.php` we'll have something like this:

```php
<?php
/* 
 * module/MyModule/config/module.config.php 
 */
return array(

    /* ... */

    'db' => array(
        'driver' => 'Mysqli', 
        'host' => 'localhost', 
        'database' => 'my_module', 
        'username' => 'my-module-user', 
        'password' => 'my-module-password'
    ),

    /* ... */

);
```

To override these default values in our application we'll create a new global configuration file for our module - `config/autoload/my-module.global.php` and put our configuration there. We don't need to list all values, just the ones to be overridden:

```php
<?php
/*
 * config/autoload/my-module.global.php
 */
return array(

    /* ... */

    'db' => array(
        'host' => 'mydb.example.org',
        'database' => 'my_app_module',
    ),

    /* ... */

);
```

It seems, that I have forgotten to configure the username and password for database access. These values should be confidential and it's not a good idea to submit them to your VCS. ZF2 proposes a way how to eliminate that risk. We'll create a `config/autoload/my-module.local.php` file and configure the credentials there:

```php
<?php
/*
 * config/autoload/my-module.local.php
 */
return array(

    /* ... */

    'db' => array(
        'username' => 'module_admin', 
        'password' => '10b4df8b1c3265ff'
    ),

    /* ... */

);
```

All `*.local.php` files in that directory should be ignored by your VCS. Actually, in the ZF2 skeleton application there is a `.gitignore` file for that. But we need to indicate somehow, that a "local" configuration is required. So we'll create a `config/autoload/my-module.local.php.dist` file with imaginary credentials:

```php
<?php
/*
 * config/autoload/my-module.local.php.dist
 */
return array(

    /* ... */

    'db' => array(
        'username' => 'module_admin', 
        'password' => 'secret'
    ),

    /* ... */

);
```

Now we can submit this file to our VCS with no worries. If someone clones the repository, he'll just copy the `*.local.php.dist` files to `*.local.php` files and alter the values appropriately.

The ZF2 module manager then makes sure, that all configuration is merged the right way - `my-module.global.php` overrides values from `module.config.php` and `my-module.local.php` overrides values from `my-module.global.php`.

So let's summarize:

* `config/application.config.php` - mostly you'll just add modules there
* `module/MyModule/config/module.config.php` - put your module-specific configuration there, but make sure not to put any environment specific values (like concrete hostname) there, use default values
* `config/autoload/my-module.global.php` - override the default values from you module with ones specific for your environment
* `config/autoload/my-module.local.php` - may contain confidential values not to be submitted to the VCS such as authentication credentials