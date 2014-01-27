---
layout: post
title: "Zend Framework 2   the Logger factory"
description: ""
category: Devel
tags: [ZF2, PHP]
---
{% include JB/setup %}

In Zend Framework 1 there was a nice factory method which allowed a logger object to be created with all its writers and filters just by passing an array or the corresponding Zend_Config value. In Zend Framework 2, there is no such method anymore. Probalby due to the effort to discourage the use of static factory methods. Anyway, it is still possible to do the same, although it is not so obvious.


According to the documentation, to create a Logger object you need to do something like this:

```php
<?php
$logger = new Zend\Log\Logger();
$writer = new Zend\Log\Writer\Stream('php://output');

$logger->addWriter($writer);
```

Or this:

```php
<?php
$logger = new Zend\Log\Logger();
$logger->addWriter('stream', null, array(
    'stream' => 'php://output'
));
```

But you can do also this:

```php
<?php
$logger = new Zend\Log\Logger(array(
    'writers' => array(
        'stream' => array(
            'name' => 'stream',
            'options' => array(
                'stream' => 'php://output'
            )
        )
    )
));
```

The constructor of the logger implements the functionality of the static factory method in ZF1. Actually, this way of logger creation is used in the LoggerServiceFactory and the LoggerAbstractServiceFactory. But both expect the logger configuration to be placed under the top-level configuration key "log". If this is the case, you can simply register the factory (in the skeleton application it is done by default) in your service manager.

Otherwise you can write your own factory class or just register a simple callback:

```php
array(
    'factories' => array(
        'My\Logger' => function ($services)
        {
            $config = $services->get('Config');
            if (! isset($config['my_namespace']['logger']) || 
                ! is_array($config['my_namespace']['logger'])) {
                throw new \RuntimeException();
            }
            
            return new Logger($config['my_namespace']['logger']);
        }
    )
);
```

But what is the right structure of the configuration? There is a simple pattern - the configuration for each entity (writer, formatter, filter) should contain a name and a list of options. Each writer can have a specific formatter and a list of filters assigned, for example:

```php
array(
    'logger' => array(
        'writers' => array(
            'stream' => array(
                'name' => 'stream',
                'options' => array(
                    'stream' => '/data/log/my.log',
                    'filters' => array(
                        'priority' => array(
                            'name' => 'priority',
                            'options' => array(
                                'priority' => 7
                            )
                        ),
                        'suppress' => array(
                            'name' => 'suppress',
                            'options' => array(
                                'suppress' => false
                            )
                        )
                    ),
                    'formatter' => array(
                        'name' => 'simple',
                        'options' => array(
                            'dateTimeFormat' => 'Y-m-d H:i:s'
                        )
                    )
                )
            )
        )
    )
);
```

The name of the writer / filter / formatter can be:

* The name under which the concrete class has been registered at the corresponding plugin manager. By default, Zend\Log\Logger uses the Zend\Log\WriterPluginManager where all writers from Zend\Log\Writer\* are registered. The same way each writer has a filter plugin manager and a formatter plugin manager.
* The name of an existing class which can be autoloaded and implements the corresponding interface (WriterInterface, FormattedInterface, etc.)

If you have custom writers / filters / formatters and you want to use the configuration as shown above, you must reference them by their class name. It is not possible to register them as plugins, because there is no way how to inject the corresponding plugin manager before the logger is constructed (in the constructor).

```php
array(
    'logger' => array(
        'writers' => array(
            'stream' => array(
                'name' => 'My\Log\Writer\CustomStream',
                'options' => array(
                    'stream' => '/data/log/my.log',
                    'filters' => array(
                        'priority' => array(
                            'name' => 'My\Log\Filter\Custom',
                            'options' => array()
                        )
                    ),
                    'formatter' => array(
                        'name' => 'My\Log\Formatter\Custom',
                        'options' => array()
                    )
                )
            )
        )
    )
);
```

There is one more issue. If all you need is simple logging to a file, with priority setting and custom formatting, the configuration is a bit too verbose. The solution might be to use a "shorthand" version such as:

```php
array(
    'logger' => array(
        'enabled' => true,
        'path' => '/data/log/my.log',
        'priority' => 4,
        'format' => '%timestamp% (%priority%): %message%'
    )
);
```

And then convert it to the "conventional" format in the factory:

```php
array(
    'factories' => array(
        'My\Logger' => function ($services)
        {
            $config = $services->get('Config');
            if (! isset($config['my_namespace']['logger']) || 
                ! is_array($config['my_namespace']['logger'])) {
                throw new \RuntimeException();
            }
            
            $shortHandConfig = $config['my_namespace']['logger'];
            $loggerConfig = array(
                'writers' => array(
                    'stream' => array(
                        'name' => 'stream',
                        'options' => array(
                            'stream' => $shortHandConfig['path'],
                            'filters' => array(
                                'priority' => array(
                                    'name' => 'priority',
                                    'options' => array(
                                        'priority' => $shortHandConfig['priority']
                                    )
                                ),
                                'suppress' => array(
                                    'name' => 'suppress',
                                    'options' => array(
                                        'suppress' => ! $shortHandConfig['enabled']
                                    )
                                )
                            ),
                            'formatter' => array(
                                'name' => 'simple',
                                'options' => array(
                                    'format' => $shortHandConfig['format']
                                )
                            )
                        )
                    )
                )
            );
            
            return new Logger($loggerConfig);
        }
    )
);
```
