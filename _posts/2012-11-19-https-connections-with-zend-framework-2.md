---
layout: post
title: "HTTPS Connections With Zend Framework 2"
description: ""
category: Devel
tags: [ZF2, PHP]
---
I suppose, that most of you know, how bad is to skip peer verification when accessing resources through SSL. There is a nice article dealing with the topic from the PHP perspective - [Insufficient Transport Layer Security (HTTPS, TLS and SSL)](http://phpsecurity.readthedocs.org/en/latest/Transport-Layer-Security-%28HTTPS-SSL-and-TLS%29.html). Generally, the purpose of SSL is to secure the connection itself through encryption and also to provide authentication of the communicating peers. That means, data are not only sent through an encrypted channel, but also to the right target. To verify the remote host, you need to check, if the certificate it presents to you is signed by a trusted certification authority.

In [Zend Framework 2](http://framework.zend.com/) there is a powerful HTTP client class, which implements almost anything you'll need when processing HTTP requests. It supports different implementations through the adapter pattern. Each adapter has its own set of options, that correspond to the underlying implementation.

The default adapter is the [Socket adapter](http://framework.zend.com/manual/2.2/en/modules/zend.http.client.adapters.html#the-socket-adapter), which uses the [PHP stream API](http://php.net/manual/en/book.stream.php). The stream API uses the SSL context when connecting through HTTPS. Some of the [SSL context options](http://php.net/manual/en/context.ssl.php) can be set as adapter options, but not all of them. So generally, it's better to set the context options directly with the [setStreamContext()](http://apigen.juzna.cz/doc/zendframework/zf2/class-Zend.Http.Client.Adapter.Socket.html#_setStreamContext) adapter method.

The most essential options are shown in the example below:

```php
<?php
$client = new Http\Client();

$adapter = new Zend\Http\Client\Adapter\Socket();
$adapter->setStreamContext(array(
    'ssl' => array(
        'verify_peer' => true,
        'allow_self_signed' => false,
        'cafile' => '/etc/ssl/certs/ca-bundle.pem',
        'verify_depth' => 5,
        'CN_match' => 'example.org'
    )
));

$client->setAdapter($adapter);
```

One of the problems with the Socket adapter is the CN match verification. The configuration itself is not very straightforward. You need to set the `CN_match` context option with the proper hostname before each request. But what is even worse, the Subject Alternative Names (SANs) are ignored, which may be a problem if you have one certificate for multiple domains.

Therefore, the [cURL adapter](http://framework.zend.com/manual/2.2/en/modules/zend.http.client.adapters.html#the-curl-adapter) seems to be a better choice, as it provides CN match verification with SANs support out of the box, although [cURL options](http://php.net/manual/en/function.curl-setopt.php) aren't too intuitive as well. You need to set `CURLOPT_SSL_VERIFYPEER`to true (or 1) and `CURLOPT_SSL_VERIFYHOST` to 2. Be careful not to set `CURLOPT_SSL_VERIFYHOST` to true, because this will be converted to 1, which means that the hostname in the CN won't be verified.

cURL example:

```php
<?php
$client = new Http\Client();

$adapter = new Zend\Http\Client\Adapter\Curl();
$adapter->setOptions(array(
    'curloptions' => array(
        CURLOPT_SSL_VERIFYPEER => true,
        CURLOPT_SSL_VERIFYHOST => 2,
        CURLOPT_CAINFO => '/etc/ssl/certs/ca-bundle.pem'
    )
));

$client->setAdapter($adapter);
```

When verifying a remote host, you need to provide one or more certificates of trusted CAs. The easier option is to have them in a single file. Then you can use the cafile stream context option (Socket) or the `CURLOPT_CAINFO` option (cURL). If you have multiple CA certificate files in a directory, you have to use the capath (Socket) or the `CURLOPT_CAPATH` (cURL) option, which has to point to that directory. The certificate files need to be properly hashed. That means - each certificate file should be referenced by a symbolic link, whose file name is derived from the hash value of the corresponding certificate. For example:

```
$ openssl x509 -hash -noout -in ca-cert.pem
9df51c42
$ ln -s ca-cert.pem 9df51c42.0
```