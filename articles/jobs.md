---
title: "Quartzite, a powerful Clojure scheduling library: defining jobs"
layout: article
---

## About this guide

This guide covers:

 * How to define periodically executed jobs
 * Using job keys to identify jobs
 * How to submit jobs for execution
 * How to pause and resume jobs
 * How to remove jobs from the scheduler
 * Using job contexts and data maps

Although many examples won't make much sense without demonstrating the use of *triggers*, this guide will focus more on jobs and
related scheduler operations, while the dedicated [guide on triggers](/articles/triggers.html) will focus more on using various
kinds of schedules.

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/welle.docs).



## What version of Quartzite does this guide cover?

This guide covers Quartzite `1.0.0-rc6`.


## Overview

Quartz separates operations that are performed (called jobs) from rules according to which they are performed (triggers). Triggers combine execution schedule
(for example, "every 4 hours" or "at noon every Friday"), date/time when execution starts and ends, operation priority and a few other things.

A job may have multiple triggers associated with them (although it is common to have just one). Both jobs and triggers have identifiers (called "keys")
that you use to manage them, for example, unschedule, pause or resume.


## Defining Quartz jobs with a Clojure DSL

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


## Using keys to identify jobs

In order to pause or completely remove a job from the scheduler, there needs to be a way to identify it. Job identifiers are called "keys". A key consists
of a string identifier and an (optional) group. To instantiate keys, use `clojurewerkz.quartzite.jobs/key` function:

{% gist b7a5152675ad0fb02ecb %}

When group is not specified, the default group is used. It is common to use groups to "namespace" executed jobs, for example, to separate operations
that perform periodic data aggregation from those that generate invoices.



## Using Quartz jobs contexts and data maps

Many jobs will need some kind of context to carry out their duties. For example, an aggregation job associated with a particular account will need
that account's id in order to load it from a data store. A job that involves retrieving Web pages may need the URL to use and so on.

Quartz (and in turn, Quartzite) allow jobs and triggers to be given a *job detail map*. It is an object that is effectively a map. Quartzite
provides convenience functions to convert Clojure maps to job detail maps and vice versa.

TBD



## Submitting jobs for execution

TBD


## Pausing and resuming jobs

TBD


## Completely removing jobs from the scheduler

TBD



## What to read next

The documentation is organized as a number of guides, covering all kinds of topics.

We recommend that you read the following guides first, if possible, in this order:

 * [Defining triggers and schedules](/articles/triggers.html)
 * [Scheduling, unscheduling and pausing jobs](/articles/unscheduling_and_pausing.html)
 * [Querying the scheduler](/articles/querying.html)
 * [Using persistent stores for scheduler state](/articles/persistent_quartz_stores.html)
 * [Using Quartz plugins](/articles/quartz_plugins.html)


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Quartzite mailing list](https://groups.google.com/forum/#!forum/clojure-quartz)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the documentation better.
