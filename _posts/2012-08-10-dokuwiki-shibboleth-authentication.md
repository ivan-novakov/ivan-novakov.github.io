---
layout: post
title: "DokuWiki Shibboleth Authentication Backend updated and moved to GitHub"
description: ""
category: Devel
tags: [ZF2, PHP, Shibboleth]
---
Few years ago I wrote a [Shibboleth authentication backend](http://blog.debug.cz/2012/03/dokuwiki-shibboleth-authentication.html) for [DokuWiki](https://www.dokuwiki.org/dokuwiki). So far it worked well, but recently I needed some additional features, so I made few modifications and enhancements:

* moved the project to [GitHub](https://github.com/ivan-novakov/dokushib)

* changed the license from the restrictive GPL2 to FreeBSD

* added an option to use the DokuWiki session instead of the Shibboleth session - the Shibboleth session is checked only upon login

* the authentication backend and the login plugin are now in one single package

* improved example configuration - better formatting and comments, you can edit it, put it in the conf/ directory and simply include it in your local.php file

* refactored the code to be more PHP 5.x compliant

You can get the code from the [GitHub repository](https://github.com/ivan-novakov/dokushib).
