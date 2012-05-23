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

This guide covers Quartzite `1.0.0-rc4`.


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

    [clojurewerkz/quartzite "1.0.0-rc4"]

### With Maven

    <dependency>
      <groupId>clojurewerkz</groupId>
      <artifactId>quartzite</artifactId>
      <version>1.0.0-rc4</version>
    </dependency>

It is recommended to stay up-to-date with new versions. New releases and important changes are announced [@ClojureWerkz](http://twitter.com/ClojureWerkz).



## Initializing the scheduler

TBD



## Jobs and triggers.

TBD


## Defining jobs.

TBD


## Using simple periodic schedules

TBD


## Scheduling jobs for execution

TBD


## Unscheduling jobs

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
