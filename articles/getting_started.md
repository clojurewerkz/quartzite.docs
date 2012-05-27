---
title: "Getting Started"
layout: article
---

## About this guide

This guide combines an overview of Quartzite with a quick tutorial that helps you to get started with it.
It should take about 10 minutes to read and study the provided code examples. This guide covers:

 * Features of Quartzite
 * Clojure version requirement
 * How to add Quartzite dependency to your project
 * Basic operations (initializing the scheduler, scheduling and unscheduling jobs)
 * Overview of how persistent schedule stores are used with Quartzite

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/welle.docs).


## What version of Quartzite does this guide cover?

This guide covers Quartzite `1.0.0-rc5`.


## Quartzite Overview

Quartzite is an expressive scheduling Clojure library built on top of the [Quartz Scheduler](http://quartz-scheduler.org). It supports all Quartz 2.1.x features,
provides a clean Clojure DSL and is well maintained.


### What Quartzite is not

Quartzite is not a replacement for Quartz, instead, Quartzite builds on top of it. Quartzite is not a standalone tool like Cron, it is a library that
applications use to programmatically schedule operations, unschedule them, calculate schedules using data from various data sources. Quartzite benefits
from the extensibility of Quartz, for example, pluggable persistent schedule storage engines.

### Project Maturity

Quartzite is in the release candidates for 1.0. The API is locked down and new features are introduced only if they are really necessary. We expect
1.0 to be released after most of the documentation guides are complete.


## Supported Clojure versions

Quartzite is built from the ground up for Clojure 1.3 and later.


## Adding Quartzite Dependency To Your Project

### With Leiningen

    [clojurewerkz/quartzite "1.0.0-rc5"]

### With Maven

    <dependency>
      <groupId>clojurewerkz</groupId>
      <artifactId>quartzite</artifactId>
      <version>1.0.0-rc5</version>
    </dependency>

It is recommended to stay up-to-date with new versions. New releases and important changes are announced [@ClojureWerkz](http://twitter.com/ClojureWerkz).


## Overview

Quartzite is a scheduling library. It lets you execute operations ("jobs") on specific schedules and with desired error handling. Unlike many other
similar systems (cron being one of the oldest and probably the most widely known), Quartzite is not a standalone tool. Instead, it is an API that
applications use to integrate scheduling functionality.


## Initializing the scheduler

Quartzite is built on top Quartz. In Quartz, a scheduler is a temporary database of jobs, schedules and other related information. Before jobs can be submitted for execution,
the scheduler has to be initialized with the `clojurewerkz.quartzite.scheduler/initialize` function:

{% gist aebee54cde1563207b19 %}

This function starts a an instance of Quartz scheduler that maintains a thread pool of non-daemon threads. Using this function in
the top-level namespace code is a **bad idea**: it will cause Leiningen tasks like `lein jar` to hang forever because Quartz threads will prevent
JVM from exiting.

When Quartzite scheduler is initialized, it does not trigger jobs until it is started using `clojurewerkz.quartzite.scheduler/initialize`. Very often this will happen on application start up:

{% gist 73a4649e06699246ff82 %}



## Jobs and triggers.

Quartz separates operations that are performed (called jobs) from rules according to which they are performed (triggers). Triggers combine execution schedule
(for example, "every 4 hours" or "at noon every Friday"), date/time when execution starts and ends, operation priority and a few other things.

A job may have multiple triggers associated with them (although it is common to have just one). Both jobs and triggers have identifiers (called "keys")
that you use to manage them, for example, unschedule, pause or resume.


## Defining jobs.

Jobs in Quartz are objects that implement `org.quartz.Job`, a single function interface. One way to define a job is to define a record
that implements that interface:

{% gist 65ff3d357ae69416287d %}

This does not look very Clojuric, does it. Because jobs are single method interfaces, it makes perfect sense to use Clojure functions as jobs.
Unfortunately, due to certain Quartz implementation details and the way Clojure loads generated classes, many approaches to using
functions do not work.

Quartzite provides a macro that makes defining jobs more concise but avoids limitations of using proxies and reification. The macro
is `clojurewerkz.quartzite.jobs/defjob`:

{% gist 07ac8e17a0d1181c0ba1 %}

These examples demonstrate defining the "executable part" of a job. As we've mentioned before, to make it possible to manage jobs and triggers,
Quartz requires them to have identities. To define a complete job that can be submitted for scheduling, you use a DSL in the `clojurewerkz.quartzite.jobs`
namespace:

{% gist 8095389fc6dbe4ec224b %}

`clojurewerkz.quartzite.jobs/key` function can be used with any other function that accepts job keys.

Now that we know how jobs are defined, lets move on to triggers.


## Using simple periodic schedules

Triggers are defined using another DSL, this time from the `clojurewerkz.quartzite.triggers` namespace. Lets define a trigger that fires
(executes its associated job) 10 times every 200 ms, starting immediately:

{% gist cfa1d575050843aeca35 %}

So how triggers are defined is quite similar to how jobs are defined. Quartz provides several types of execution schedules
and Quartzite supports all of them in the DSL. A couple more schedule types will be demonstrated later in this guide.

Now that we know how to define jobs and triggers, lets schedule them for execution.


## Scheduling jobs for execution

`clojurewerkz.quartzite.scheduler/schedule` submits a job and a trigger associated with it for execution:

{% gist de06e30b0b188bfce5f8 %}

If the scheduler is started, execution begins according to the start moment of the submitted trigger. In the example above, the trigger
will fire 10 times every 200 ms and expire after that. Expired triggers do not execute associated jobs.


## Unscheduling jobs

{% gist 78e43f36ea261657b7c5 %}

TBD


## Using Cron expression schedules

TBD


## Using calendar schedules

TBD


## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [Scheduler initialization](/)
 * [Defining jobs](/)
 * [Defining triggers and schedules](/)
 * [Scheduling, unscheduling and pausing jobs](/)
 * [Querying the scheduler](/)
 * [Using persistent stores for scheduler state](/)
 * [Using Quartz plugins](/)


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Quartzite mailing list](https://groups.google.com/forum/#!forum/clojure-quartz)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.
