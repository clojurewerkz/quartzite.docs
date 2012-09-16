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

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/quartzite.docs).



## What version of Quartzite does this guide cover?

This guide covers Quartzite `1.0.x`.


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



## Jobs contexts and job data maps

Many jobs will need some kind of context to carry out their duties. For example, an aggregation job associated with a particular account will need
that account's id in order to load it from a data store. A job that involves retrieving Web pages may need the URL to use and so on.

When Quartz executes a job, it will pass it a **job context** object that among other things, includes arbitrary data that the job needs.

Lets take a look at a simplest Quartzite job possible:

{% gist 1fb0eb95d00cf2030db3 %}

It takes the aforementioned **job context** which is an instance of [JobExecutionContext](http://quartz-scheduler.org/api/2.1.5/org/quartz/JobExecutionContext.html).
The job execution context you can retrieve **job data map**. Quartzite offers a function that returns **job data map** as an immutable Clojure map:

{% gist 1fc14e13304951430b2b %}

Job data is optional and can be added via the job definition DSL:

{% gist e565e0520803ef7f8d3f %}


## Scheduling jobs for execution

`clojurewerkz.quartzite.scheduler/schedule` submits a job and a trigger associated with it for execution:

{% gist de06e30b0b188bfce5f8 %}

If the scheduler is started, execution begins according to the start moment of the submitted trigger. In the example above, the trigger
will fire 10 times every 200 ms and expire after that. Expired triggers do not execute associated jobs.


## Unscheduling jobs

To unschedule an operation, you unschedule its trigger (or several of them) using `clojurewerkz.quartzite.scheduler/unschedule-job`
and `clojurewerkz.quartzite.scheduler/unschedule-jobs` functions:

{% gist 78e43f36ea261657b7c5 %}

Please note that `unschedule-job` takes a *trigger* key. `clojurewerkz.quartzite.scheduler/unschedule-jobs` works the same way but
takes a collection of keys.

There are other functions that delete jbos and all their triggers, pause execution of triggers and so on. They are covered in the
[Scheduling, unscheduling and pausing jobs](/articles/unscheduling_and_pausing.html) guide.


## Pausing and resuming jobs

Jobs can be paused and resumed. Pausing a job pauses all of its triggers so the job won't be executed but is not removed from
the scheduler. To pause a single job, use `clojurewerkz.quartzite.scheduler/pause-job` and pass it the job's key:

{% gist fef827c935c46a14c4d6 %}

`clojurewerkz.quartzite.scheduler/pause-jobs` will pause one or several groups of jobs by pausing their triggers. What groups
are paused is determined by the *group matcher*, instantiated via Java interop:

{% gist 5d40ead93175447be57e %}

In addition to the exact matcher, there are several other matchers available:

{% gist dcd5d0191fa4dea461d3 %}

Resuming a job makes all its triggers fire again. `clojurewerkz.quartzite.scheduler/resume-job` is the function that does
that for a single job:

{% gist 71a9fc5637912b1ec0af %}

`clojurewerkz.quartzite.scheduler/resume-jobs` resumes one or more job groups using the already covered group matchers:

{% gist 19e7d950de3c279b8756 %}

Finally, `clojurewerkz.quartzite.scheduler/pause-all!` and `clojurewerkz.quartzite.scheduler/resume-all!` are functions
that pause and resume *the entire scheduler*. Use them carefully. Both take no arguments.

### Jobs and Trigger Misfires

A misfire occurs if a persistent trigger "misses" its firing time because of being paused, the scheduler being shutdown, or because there are no
available threads in Quartz's thread pool for executing the job. When a job is resumed, if any of its triggers have missed
one or more fire-times, the trigger's misfire instruction will apply.

Misfires are covered in the [triggers guide](/articles/triggers.html).


## Completely removing jobs from the scheduler

It is possible to completely remove a job from the scheduler. Doing so will also remove *all the associated triggers*. The job
will never be executed again (unless it is re-scheduled). `clojurewerkz.quartzite.scheduler/delete-job` deletes
a single job by job key:

{% gist 3fcc732121cedf45ea76 %}

while `clojurewerkz.quartzite.scheduler/delete-jobs` removes multiple jobs and takes a collection of keys:

{% gist f69e11989ec9fd8f72ad %}



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
