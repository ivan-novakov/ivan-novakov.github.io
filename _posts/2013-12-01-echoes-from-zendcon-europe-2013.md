---
layout: post
title: "Echoes from ZendCon Europe 2013"
description: ""
category: Devel
tags: [ZF2, PHP, conference]
---

ZendCon is probably the biggest and most important conference in the PHP world. And for the first time it took place in Europe - Paris (18-20 November). Many brilliant and famous engineers, developers, speakers, entrepreneurs had their presentation there. Also some of the most important people  from Zend (“the PHP company”) were there to present new (and not so new) ideas, tools, best practices and to answer questions either from the stage in front of the audience or by the coffee stand in private.

My impression of the conference as a whole is that the API-centric and mobile-first principles will have growing importance in the future. That seems to be the main direction Zend is wishing to follow. At the same time automated continuous deployment is considered to be of great value in the software engineering process. And let’s not forget the “clouds”. They have been around for some time, but more and better integrated tools are being introduced either by the big players such as Google, Microsoft, IBM or by smaller but specialized vendors.

## My notes from some of the presentations

### Relax with ZF2: RESTful APIs
by [Matthew Weier O’Phinney](https://twitter.com/mwop)

* Writing RESTful applications the right way is hard.
* There are many things you need to take care of - proper request routing, headers, response status codes, data formatting, error handling etc.
* Most of this stuff can be automated - introducing [Apigility](http://www.apigility.org/), a [ZF2](http://framework.zend.com/) based open source tool for creating and managing RESTful APIs.
* Apigility itself is an API based web application with AngularJS-enhanced frontend.
* It can create APIs which are “connected” to an existing service layer or directly to a database.
* It handles the “view-controller” part of the RESTful application - routing, content negotiation, error handling etc.
* Data are returned in JSON implementing the HAL specification.
* Exceptions from the service layers are caught and the corresponding error response is being generated using the API Problem specification.
* Apigility also supports API versioning - the client may request for a specific version either through the request URL or by setting the corresponding “Content-Type” header.
* Apigility has currently a pre-release status, but the 1.0 version is expected soon - probably in Q1 2014.

### Introducing Dependency Injection
by [Rob Allen](https://twitter.com/akrabat)

* Tightly coupled code brings troubles.
* Dependency injection enables loose coupling of the code and thus improving maintainability, extensibility and testability
* Inversion of control - object creation is handled by an “external” entity, objects are not created inside of another object, first they are created and then “injected” into the dependent object.
* Using dependency injection increases the verbosity of the code. It may even introduce duplicities, if we need to instantiate the same type of object in different parts of the code.
* Dependency injection container - a special object, that automates creation of objects and their dependencies. It also separates configuration from construction. DICs may be explicit, when we write manually the code responsible for object creation, or automated, when it resolves dependencies automatically, for example by type-hinted constructor or method arguments.
* Available DICs:
    * Pimple
    * Dice
    * SymfonyContainer
    * Zend\Di and Zend\ServiceManager
 
### Scenario Driven API Design
by [Ivo Jansch](https://twitter.com/ijansch)
 
* REST is not just CRUD over a database.
* If a RESTful API is just a thin layer over the database, it means that the client is supposed to implement most of the business logic.
* There are some problems with that approach:
    * performance - for a single action the client may need multiple requests to the backend 
    * duplicity - if there is need for another client (for example a mobile application), you’ll have to implement the business logic twice
* Instead of implementing a low level “thin” API exposing your “raw” model, analyze the possible scenarios, figure out how the API is going to be used and then build your API using higher level abstractions.
* In other words - put your business logic on the API level (smart APIs, dumb clients).
* An API may not be 100% RESTful, the usability aspect is far more important, so if the usage scenario requires it, you may “break” the rules.
 
### Refactoring
by [Adam Culp](https://twitter.com/adamculp)
 
* Refactoring - changing a program’s source code without modifying its external functional behavior.
* Reasons for refactoring - improving design, flexibility, maintainability, readability etc.
* Do not add functionality during refactoring - refactor first and then add functionality, then you can refactor again and so on.
* Rewrite vs. refactor
    * While refactoring we modify small pieces of code and gradually improve the program, thus saving time and resources.
    * Rewriting a program from scratch requires much more time and resources and the outcome may be uncertain.
* Use some kind of source control (Git, SVN, etc.) - allows to record each step and rollback, if there are problems.
* Proper refactoring is not possible without testing. Basic steps:
    * Ensure tests pass.
    * Refactor.
    * Ensure tests still pass.
    * Add more tests or update the existing ones, if the program modifications require it.
    * Repeat.
* Optimization and refactoring are different actions - refactor first, the optimize later if needed.
* Recommended reading - “Refactoring: Improving The Design of Existing Code” book, by Martin Fowler.

### Building Scalable PHP Applications Using Google’s App Engine
by [Amy Unruh](https://plus.google.com/+AmyUnruh)
 
* Benefits of the platform:
    * scale on demand
    * global availability
    * analytics
    * easy development - free start, local development environment, service abstractions
    * easy management
* PHP runtime - version 5.4.19, limited number of modules
* APIs and services
    * Data - Cloud Datastore, Cloud SQL, Cloud Storage
    * Memcache
    * Task Queue - for running asynchronous tasks
    * User service - provides authentication framework
    * Mail - allows sending emails
    * …
* PHP SDK
    * a local development environment that simulates App Engine services
    * allows to deploy applications to App Engine
    * available for Windows, Mac OS X and Linux
* Migrating an application to App Engine
    * create the application’s App Engine configuration (app.yaml)
    * modify the application to use the App Engine services such as the streams API, Mail API, Task Queues etc.
    * migrate data (MySQL → Cloud SQL, NoSQL → Cloud Datastore)
* Task Queues
    * used for running tasks asynchronously in the background
    * automatically scaled processing capacity
    * task handler scripts - contain the code to be executed, exposed on a specific URL
    * multiple queues may be used
 
### LEVEL UP! Migrating your ZF1 app to ZF2
by [Gary Hockin](https://twitter.com/GeeH)
 
* Two possible strategies:
    * Strangler - modify the code little by little by replacing parts of the ZF1 app with ZF2 components
    * Big Bang (recommended) - rewrite the entire app, refactor using ZF2 components - should be much easier if you have a service layer
* New things in ZF2
    * Modules - unlike modules in ZF1, modules in ZF2 are really independent and reusable pieces of code
    * Different project layout - modules are first-class citizens, so the modules’ directory is placed in the root directory, all the module’s controllers, models, forms and other code is placed in one PSR compliant sub directory of the particular module.
    * Views - separated data (ViewModel) and separated rendering layer (ViewRenderer)
    * ServiceManager - handles (lazy) object creation and dependency injection
    * EventManager - allows objects to trigger events and if there are callbacks registered for that event, they are executed.
 
### Doctrine 2 and Zend Framework 2
by [Marco Pivetta](https://twitter.com/Ocramius)
 
* Doctrine ORM is an Object Relational Mapper inspired by Hibernate and based on DBAL (DataBase Abstraction Layer).
* In ORM you work with objects backed by a persistence layer, you shouldn’t be concerned how the persistence is implemented or what kind of database is used.
* Entities - simple objects with identifiers, contain data and no business logic, just simple checks.
* The object manager - responsible for entity persistence, it can fetch, save, update and delete entities.
* Associations - an entity may be associated with other entities in different relations (one to one, one to many, many to many). These relations are resolved by the object manager automatically.
* Collections - OOP API for array-like data structures
* Criteria API - searching and filtering abstraction.
* Doctrine integration into ZF2
    * Doctrine ZF2 modules (DoctrineModule, DoctrineORMModule, DoctrineMongoODMModule)
    * Doctrine configuration is placed in the module.config.php file.
    * Doctrine services (object manager, entity repository) are available through the ServiceManager


## Links

* [ZendCon Europe official site](http://europe.zendcon.com/)
* [ZendCon Europe event on Joind.in](https://joind.in/event/view/1515) (slides and ratings)

