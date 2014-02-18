---
layout: post
title: "DokuWiki Shibboleth Authentication Plugin"
description: ""
category: Devel
tags: [ZF2, PHP]
---
The new [DokuWiki](https://www.dokuwiki.org/) version 2013-05-10 “Weatherwax” introduced new approach to modular authentication. While the older versions used authentication backends, the new version makes use of its flexible plugin system and introduces a new plugin type - the [authentication plugin](https://www.dokuwiki.org/devel:auth_plugins). Actually, it is very similar to the authentication backend, but as a plugin it provides all the benefits of the plugin system - it can be installed via DokuWiki administration, it can be configured with the configuration manager, etc.

That was an impulse for me to rewrite my [Shibboleth authentication backend](https://github.com/ivan-novakov/dokushib) from scratch and implement it as a plugin. The old backend required a simple action plugin to intercept the login action and redirect the browser to the Shibboleth login handler. So it was necessary to install both the backend and the plugin. Now, when the authentication is done via plugins, only one plugin is required. The plugin system allows combination of different plugins in a single plugin bundle installed as one.

You can get the plugin from the new [GitHub repository](https://github.com/ivan-novakov/dokuwiki-shibboleth-auth). See the README for instructions how to install it.

Links:

* [GitHub repository](https://github.com/ivan-novakov/dokuwiki-shibboleth-auth)
* [issues](https://github.com/ivan-novakov/dokuwiki-shibboleth-auth/issues)
* [DokuWiki plugin page](https://www.dokuwiki.org/plugin:authshibboleth)
* [the old backend](https://github.com/ivan-novakov/dokushib)
