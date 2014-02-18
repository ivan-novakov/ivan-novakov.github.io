---
layout: post
title: "Shibboleth Authentication Adapter for the Zend Framework"
description: ""
category: Devel
tags: [ZF1, PHP]
---
I have been running some sites under a [Shibboleth Service Provider](http://www.shibboleth.net/) for quite a long time. In some cases I had to modify applications written in PHP to be able to use Shibboleth authentication. But now I reached the point, when I had to implement Shibboleth authentication in my own application. Since I have been using Zend Framework for years, the natural way to achieve that was implementing a Shibboleth authentication adapter for the `Zend_Auth` component.


Actually, the task is quite simple, because all the required information is contained in the environment variables. The only "challenge" is to make sure the adapter is as generic and flexible as possible, so it can be deployed in various environments and applications. The configuration options for the adapter mostly specify the names of the attributes containing relevant information and possibly a mapping between the attribute names and the local names. Typical usage:

```php
<?php
$auth = Zend_Auth::getInstance();

$authAdapter = new ShibbolethAdapter(array(
        'identityVar' => 'id', 
        'attrMap' => array(
            'uid' => 'id', 
            'cn' => 'name',
            'mail' => 'email'
        )
));

$result = $auth->authenticate($authAdapter);
```

If the authentication is successfull, the result variable will contain an array of attributes with re-mapped indexes, for example:

```php
<?php
array(
    'id' => 'ivan',
    'name' => 'Ivan Novakov',
    'email' => 'ivan.novakov[at]debug.cz'
);
```

Of course, if your application requires the user identity in another format or even as an object (which is quite usual :-)), it is possible to change that behaviour by subclassing the adapter and implementing your own `_createResult()` method. The same method may also implement additional authentication checks. For example, many applications, which use Shibboleth authentication, have a local user database and the user identity received from Shibboleth has to be validated against it.

See the complete [source code](https://github.com/ivan-novakov/zf-shib-auth) for more details. Please, note, that this is a simple piece of code, the main purpose of which is to inspire. Feel free to use it and modify it to suit your needs.

```php
<?php
/**
 * Authentication adapter for the Zend Framework authentication component Zend_Auth.
 */
class ShibbolethAdapter implements Zend_Auth_Adapter_Interface
{

    /**
     * The configuration object.
     * 
     * @var Zend_Config
     */
    protected $_config = NULL;

    /**
     * Default values for all the options.
     * 
     * @var array
     */
    protected $_defaultOptions = array(
        
        // an attribute prefix to apply to all the attributes
        'attrPrefix' => '', 
        // the attribute value separator, if the attribute has multiple values
        'attrValueSeparator' => ';', 
        
        // the attribute holding the session string
        'sessionIdVar' => 'Shib-Session-ID', 
        // the attribute containing the entityID of the IdP
        'idpVar' => 'Shib-Identity-Provider', 
        // the attribute containing the application ID (referenced in shibboleth2.xml)
        'appIdVar' => 'Shib-Application-ID', 
        // the attribute conatining  the time of the authentication
        'authInstantVar' => 'Shib-Authentication-Instant', 
        // the attribute containing the authentication context
        'authContextVar' => 'Shib-AuthnContext-Decl', 
        
        // the attribute used to determine the identity of the user
        'identityVar' => 'uid', 
        
        // an array, which maps Shibboleth attributes to local user attributes
        'attrMap' => array(
            'eppn' => 'uid', 
            'cn' => 'cn', 
            'mail' => 'email'
        )
    );

    /**
     * Array of environment variables.
     * 
     * @var array
     */
    protected $_env = array();


    /**
     * Constructor.
     * 
     * @param array $config
     * @param array $env
     */
    public function __construct (Array $config = array(), Array $env = NULL)
    {
        $this->_config = new Zend_Config($config + $this->_defaultOptions);
        
        if (! $env) {
            $env = $_SERVER;
        }
        $this->_env = $env;
    }


    /**
     * Implementation of the authenticate() method.
     * 
     * @see Zend_Auth_Adapter_Interface::authenticate()
     */
    public function authenticate ()
    {
        if (! $this->_isSession()) {
            return new Zend_Auth_Result(Zend_Auth_Result::FAILURE, NULL, array(
                'no_session'
            ));
        }
        
        $userAttrs = $this->_extractAttributes();
        if (! isset($userAttrs[$this->_config->identityVar])) {
            return new Zend_Auth_Result(Zend_Auth_Result::FAILURE_IDENTITY_NOT_FOUND, NULL, array(
                'no_identity'
            ));
        }
        
        if (is_array($userAttrs[$this->_config->identityVar])) {
            return new Zend_Auth_Result(Zend_Auth_Result::FAILURE_IDENTITY_AMBIGUOUS, NULL, array(
                'multiple_id_attr_value'
            ));
        }
        
        return $this->_createResult($userAttrs);
    }


    /**
     * Convenience method for creating a positive result with user identity.
     * 
     * Can be overrided in subclasses in case the user identity is required in another format 
     * (an object for instance).
     * 
     * @param array $userAttrs
     */
    protected function _createResult (Array $userAttrs)
    {
        return new Zend_Auth_Result(Zend_Auth_Result::SUCCESS, $userAttrs);
    }


    /**
     * Returns the array of available attributes.
     * 
     * @return array
     */
    protected function _extractAttributes ()
    {
        $attrs = array();
        foreach ($this->_config->attrMap->toArray() as $srcIndex => $dstIndex) {
            if ($value = $this->_getEnv($srcIndex)) {
                $values = explode($this->_config->attrValueSeparator, $value);
                if (count($values) > 1) {
                    $attrs[$dstIndex] = $values;
                } else {
                    $attrs[$dstIndex] = $value;
                }
            }
        }
        
        return $attrs;
    }


    /**
     * Returns true, if there is an existing Shibboleth session.
     * 
     * @return bool
     */
    protected function _isSession ()
    {
        return ($this->_getSession());
    }


    /**
     * Returns the current Shibboleth session string.
     * 
     * @return string
     */
    protected function _getSession ()
    {
        return $this->_getEnv($this->_config->sessionIdVar);
    }


    /**
     * Returns the required environment variable.
     * 
     * @param string $index
     * @return string
     */
    protected function _getEnv ($index)
    {
        $index = $this->_config->attrPrefix . $index;
        
        if (isset($this->_env[$index])) {
            return $this->_env[$index];
        }
        
        return NULL;
    }
}
```