---
layout: post
title: "OAuth2 / OpenID Connect Library for PHP / ZF2"
description: ""
category: Devel
tags: [ZF2, PHP]
---
I'm involved in federated identity management, delegated authorization and RESTful web services. So it was natural that I chose to adopt the [OAuth2 framework](http://oauth.net/2/) and its more specific "brother" - [OpenID Connect](http://openid.net/connect/). There are already some client implementations of OpenID Connect and even more implementations of the OAuth2 specification. But I had my own reasons, why I wrote [my own implementation](https://github.com/ivan-novakov/php-openid-connect-client):

* I use [PHP](http://php.net/), [Zend Framework 2](http://framework.zend.com/) and [composer](http://getcomposer.org/) and I'm used to that :)

* instead of a monolitic client implementation I need a library/framework which provides tools and building blocks for creating clients for different use cases

* OpenID Connect is not ready yet, the specs are being changed and it is easier for me to modify and adapt my own implementation

* actually, I started writing a simple client to test my server implementation, but finally it grew up to a whole library :)

In the code I tried to respect the dependency injection paradigm together with the single responsibility principle and good testability. Dependencies may be injected or created implicitly in a "lazy" manner (when they are needed). As a result, the code is fragmented into numerous objects and it may need a bit more writing to tie them together (if you do not use the implicit values). That can be solved by writing a facade such as the `InoOicClient\Flow\Basic` object, which accepts a simple configuration array and does all the initialization inside.

I successfully tested the library against [Google](https://developers.google.com/accounts/docs/OAuth2Login) and [Github](http://developer.github.com/v3/oauth/), but probably more identity providers "work" out of the box. The source repository contains simple demos, but I'm planning to write more user-frienldy ones.

The library cannot be recommended for production use yet though. It hasn't been tested enough. There are some important features from the OpenID Connect specs missing - ID token validation, tools for discovery and registration etc. Anyway, I'm planning to add them in the future releases.

More information:

* [GitHub repository](https://github.com/ivan-novakov/php-openid-connect-client) - see the README for more technical info
* [API docs](http://debug.cz/apidoc/php-openid-connect-client/)