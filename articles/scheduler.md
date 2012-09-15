---
title: "Quartzite, a powerful Clojure scheduling library: schedulers"
layout: article
---

## About this guide

This guide covers:

 * Quartz schedulers
 * Scheduler lifecycle
 * Scheduler configuration
 * Using multiple schedulers with Quartzite

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/quartzite.docs).



## What version of Quartzite does this guide cover?

This guide covers Quartzite `1.0.x`.


## Overview

Quartz scheduler is a specialized database that stores jobs and triggers and executes the jobs according to trigger schedules. It may store its state
in a durable data store (like relational databases accessible via JDBC or [MongoDB](https://github.com/michaelklishin/quartz-mongodb)).

Quartz schedulers are also *event emitters*. It is possible to register listeners for various events (such as scheduler being paused or a trigger firing).
Schedulers can be extended in using 3rd party plugins.

Quartzite assumes that most application only ever use a single scheduler.


## Quartz scheduler lifecycle

Scheduler may be in one of the following states:

 * Not started
 * Started
 * In the standby mode (paused)
 * Shut down

These is not technically an exhaustive list but it gives you a good idea.

Before jobs can be submitted for execution, the scheduler has to be initialized with the `clojurewerkz.quartzite.scheduler/initialize` function:

{% gist aebee54cde1563207b19 %}

This function starts a an instance of Quartz scheduler that maintains a thread pool of non-daemon threads. Using this function in
the top-level namespace code is a **bad idea**: it will cause Leiningen tasks like `lein jar` to hang forever because Quartz threads will prevent
JVM from exiting.

When Quartzite scheduler is initialized, it does not trigger jobs until it is started using `clojurewerkz.quartzite.scheduler/start`. Very often this will happen on application start up:

{% gist 73a4649e06699246ff82 %}

To put scheduler into the standby mode, use `clojurewerkz.quartzite.scheduler/standby` function:

{% gist a3f8e626f49544658342 %}

When in the standby mode, scheduler is not active: triggers do not fire and as a consequence, no jobs are executed.

To shut down the scheduler, use `clojurewerkz.quartzite.scheduler/shutdown`:

{% gist b91dd8789d39d90aceb8 %}


Quartzite provides several predicate functions that let you determine current state of the  scheduler:

{% gist cb8e9ebf3a316ec21211 %}


To initialize a new scheduler after the previous one has been shut down, use `clojurewerkz.quartzite.scheduler/recreate`:

{% gist b1a7d0b2cbc2e615195b %}

Typically you only will recreate schedulers in the REPL or integration tests, it is almost never used in production.


## Quartz scheduler configuration

Quartzite relies on Quartz to handle configuration. Quartz uses a property file, `quartz.properties`, that needs to be on the classpath.
With [Leiningen 2](http://leiningen.org), you specify resource (non-code) file location using the `:resource-paths` key in `project.clj`:

{% gist 72f21c60a57f08f241af %}

Here is a minimalistic example of `quartz.properties`:

{% gist 1f1a672b1a403fb03e2e %}

A more involved example may use a custom durable store and one or more plugins:

{% gist 9a449f0af083d63a5330 %}

Please refer to the [Quartz Scheduler configuration reference](http://quartz-scheduler.org/documentation/quartz-2.x/configuration/) to learn more.


## Using multiple schedulers

Quartzite was created with applications that only ever use a single scheduler (the vast majority, in our opinion). To work with multiple schedulers,
you can rebind the dynamic var Quartzite uses to store its scheduler instance, `clojurewerkz.quartzite.scheduler/*scheduler*`, using
`clojurewerkz.quartzite.scheduler/with-scheduler` convenience macro or `clojure.core/binding` directly. The var must be an atom.



## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [Defining jobs](/jobs.html)
 * [Defining triggers and schedules](/triggers.html)
 * [Scheduling, unscheduling and pausing jobs](/unscheduling_and_pausing.html)
 * [Querying the scheduler](/querying.html)
 * [Using persistent stores for scheduler state](/persistent_quartz_stores.html)
 * [Using Quartz plugins](/quartz_plugins.html)


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Quartzite mailing list](https://groups.google.com/forum/#!forum/clojure-quartz)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.
