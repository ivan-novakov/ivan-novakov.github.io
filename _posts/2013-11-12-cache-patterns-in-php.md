---
layout: post
title: "Cache Patterns In PHP"
description: ""
category: Devel
tags: [ZF2, PHP, cache]
---

No matter how skilled developer you are, sometimes you can't avoid having slow pieces of code - handling remote connections, database queries or just complicated calculations. One of the possible solutions is to implement caching. But the question is - how to do it right? In the following text I'll got through several possible ways how to implement caching in search of the best solution. I'll use the cache storage implementation from [Zend Framework 2](http://framework.zend.com/), but any other relevant implementation can be used instead.

The [ZF2 cache storage](http://framework.zend.com/manual/2.2/en/modules/zend.cache.storage.adapter.html) is instantiated like this:

```php
<?php
$cacheStorage = StorageFactory::factory(array(
    'adapter' => array(
        'name' => 'filesystem',
        'options' => array(
            'cache_dir' => '/tmp/cache',
            'ttl' => 60
        )
    ),
    'plugins' => array(
        'serializer'
    )
));
```

Let us have some example class:

```php
<?php
class SomeClass implements SomeInterface
{
    protected $label;

    public function __construct($label = 'generic')
    {
        $this->label = $label;
    }

    public function fetchSomething()
    {
        sleep(2);
        return sprintf("%s: important data", $this->label);
    }

    public function fetchAnother()
    {
        sleep(2);
        return sprintf("%s: another data", $this->label);
    }
}
```

We would like to cache the return values of the "fetch" methods.

## The simple way

The most straightforward way would be to inject the cache storage into the class and modify the methods we want to cache:

```php
<?php
use Zend\Cache\Storage\StorageInterface;

class Simple
{
    protected $label;
    protected $cacheStorage;

    public function __construct($label = 'generic')
    {
        $this->label = $label;
    }

    public function setCacheStorage(StorageInterface $cacheStorage)
    {
        $this->cacheStorage = $cacheStorage;
    }

    public function fetchSomething()
    {
        if ($this->cacheStorage && $this->cacheStorage->hasItem('something')) {
            return $this->cacheStorage->getItem('something');
        }
        sleep(2);
        $data = sprintf("%s: important data", $this->label);
        
        if ($this->cacheStorage) {
            $this->cacheStorage->setItem('something', $data);
        }
        
        return $data;
    }

    public function fetchAnother()
    {
        //...
    }
}

$obj = new Simple();
$obj->setCacheStorage($cacheStorage);

echo $obj->fetchSomething() . "\n";
```

There are two major problems in this approach:

* Unless you don't have only several methods to cache, there will be massive code multiplication.

* In many cases it may not be appropriate to modify the original class. Moreover, our object gets dependent on the cache storage object and that may break the application's layer consistency

## ZF2 cache pattern

Zend Framework 2 offers a way how to automate the caching while keeping the original object intact. You can pass your object and the cache storage to a `Zend\Cache\Pattern\ObjectCache` instance and then use it instead of the original object. The `ObjectCache` instance will take care of the caching automatically. Internally it uses the magic `__call()` method to intercept calls and serves as a proxy to the original object wrapping each delegated call with the necessary logic.

```php
<?php
use Zend\Cache\PatternFactory;

$object = new SomeClass();

$objectCache = PatternFactory::factory('object', array(
    'object' => $object,
    'storage' => $cacheStorage
));

echo $objectCache->fetchSomething() . "\n";
```

The problem here is that the proxy object is not the same object as the original object and that could possibly break some contracts like, for example in methods that use type hinting.

## Proxy implementing the target object's interface

If the original object implements an interface, we can design our proxy to implement the same interface, so it can replace the original object. For convenience I'll use the ZF2's object cache in the following example:

```php
<?php
use Zend\Cache\Pattern\ObjectCache;

class InterfaceProxy implements SomeInterface
{
    /** @var ObjectCache */
    protected $objectCache;
    
    /** @var SomeClass */
    protected $originalObject;

    public function __construct(ObjectCache $objectCache)
    {
        $this->objectCache = $objectCache;
        $this->originalObject = $this->objectCache->getOptions()->getObject();
    }

    public function fetchSomething()
    {
        return $this->objectCache->fetchSomething();
    }

    public function fetchAnother()
    {
        return $this->originalObject->fetchSomething();
    }
}
```

The "implementation" is easy - the calls we need to cache are delegated to the object cache, the others are delegated to the original object.

```php
<?php
use Zend\Cache\PatternFactory;

$object = new SomeClass();

$objectCache = PatternFactory::factory('object', array(
    'object' => $object,
    'storage' => $cacheStorage
));

$proxy = new InterfaceProxy($objectCache);

echo $proxy->fetchSomething() . "\n";
```

Here we have another problem - we need to "implement" all the methods from the interface, which is again can be a bit repetitive. But it is worth the benefits - the original object doesn't "know" anything about the cache and the proxy object mimics the behaviour of the original object, so the dependent entities don't "care" that they are using a different object. The repetitiveness may be resolved by implementing suitable generators.

## Proxy as a subclass

If the original class doesn't implement an interface, the proxy may be designed as a subclass of the original object. The solution will be similar to the previous example:

```php
<?php
use Zend\Cache\Pattern\ObjectCache;

class SubclassProxy extends SomeClass
{
    /** @var ObjectCache  */
    protected $objectCache;

    public function setObjectCache(ObjectCache $objectCache)
    {
        $this->objectCache = $objectCache;
    }

    public function fetchSomething()
    {
        if (! $this->objectCache) {
            throw new \RuntimeException('cache object not set');
        }
        return $this->objectCache->fetchSomething();
    }
}
```

```php
<?php
use Zend\Cache\PatternFactory;

$object = new SomeClass();

$objectCache = PatternFactory::factory('object', array(
    'object' => $object,
    'storage' => $cacheStorage
));

$proxy = new SubclassProxy();
$proxy->setObjectCache($objectCache);

echo $proxy->fetchSomething() . "\n";
```

Again, we'll be able to use the proxy instead of the original object. And we can "implement" only the methods that need to be cached. Still, if there are lots of them, we'll probably have to use a generator. In order not to break the LSP, we can't inject the cache object through he constructor, so we need to add checks that it has been injected.

## Smart reference proxy

A smart reference proxy _"allows you to dynamically define logic to be executed before or after any of the wrapped object's methods logic"_. I came across this kind of proxy while I was exploring Marco Pivetta's nice library for generating different kind of proxies - the [ProxyManager](https://github.com/Ocramius/ProxyManager). This is the general idea (the example is taken from the [ProxyManager documentation](https://github.com/Ocramius/ProxyManager/blob/master/docs/access-interceptor-value-holder.md)):

```php
<?php
$factory = new \ProxyManager\Factory\AccessInterceptorValueHolderFactory();

$proxy = $factory->createProxy(
    new \My\Db\Connection(),
    array('query' => function () { echo "Query being executed!\n"; }),
    array('query' => function () { echo "Query completed!\n"; })
);

$proxy->query(); // produces "Query being executed!\nQuery completed!\n"
```

Applied to our case, it could look something like this:

```php
<?php
// The "AccessInterceptorValueHolder" proxy factory
$factory = new ProxyFactory($proxyConfig);

// Cretae the proxy without any callbacks defined
$proxy = $factory->createProxy($object);

// Define a generic callback to be run before the object method
$preInterceptor = function ($proxy, $instance, $method, $params, &$returnEarly) use($cacheStorage)
{
    $key = md5(get_class($instance) . $method . serialize($params));
    if ($cacheStorage->hasItem($key)) {
        $returnEarly = true;
        return $cacheStorage->getItem($key);
    }
};

// Define a generic callback to be run after the object method
$postInterceptor = function ($proxy, $instance, $method, $params, $returnValue, &$returnEarly) use($cacheStorage)
{
    $key = md5(get_class($instance) . $method . serialize($params));
    $cacheStorage->setItem($key, $returnValue);
};

// Assignt the callbacks to different methods
$proxy->setMethodPrefixInterceptor('fetchSomething', $preInterceptor);
$proxy->setMethodSuffixInterceptor('fetchSomething', $postInterceptor);

$proxy->setMethodPrefixInterceptor('fetchAnother', $preInterceptor);
$proxy->setMethodSuffixInterceptor('fetchAnother', $postInterceptor);

// Call the methods through the proxy
echo $proxy->fetchSomething() . "\n";
echo $proxy->fetchAnother() . "\n";
```

**Note:** This is a simplified example, in "reality" you'll probably implement the cache key generation through a separate object.

This is just another way how to create a caching proxy. But the main benefit over the previous proxy examples is that you can define generic pre- and post- callbacks and attach them to multiple methods. This allows you to automate the process and you don't have to define multiple methods just to satisfy an interface. The ProxyManager generates them for you. And even better - you don't have to generate them each time. You can configure the ProxyManager to autoload previously generated proxies.

## Conclusion

Using some kind of a proxy object seems to be the right way to implement caching. And which kind of proxy you choose depends on your requirements. Using the ProxyManager might be an overkill, if your application is small. Using proxies generally introduces an additional layer to your application, so you have to figure out if it's worth the "overhead". A good way to reduce the "overhead" is to use generators. And last but not least - using proxies may be problematic in some cases. For example, as Marco Pivetta pointed out in his blog post, there are problems when trying to proxy fluent interfaces.

Anyway, I'll be glad to get some feedback and hear/read about alternative ways how to implement caching.

