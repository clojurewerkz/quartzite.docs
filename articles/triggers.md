---
title: "Quartzite, a powerful Clojure scheduling library: defining triggers and schedules"
layout: article
---

## About this guide

This guide covers:

 * How to define triggers
 * Using trigger keys to identify triggers
 * How to use various types of schedules
 * How to pass context information to executed jobs

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/quartzite.docs).



## What version of Quartzite does this guide cover?

This guide covers Quartzite `1.0.0-rc6`.


## What Quartz triggers

Quartz separates operations that are performed (called jobs) from rules according to which they are performed (triggers). Triggers combine execution schedule
(for example, "every 4 hours" or "at noon every Friday"), date/time when execution starts and ends, operation priority and a few other things.

A job may have multiple triggers associated with them (although it is common to have just one). Both jobs and triggers have identifiers (called "keys")
that you use to manage them, for example, unschedule, pause or resume.


## Defining Quartz triggers with a Clojure DSL

Triggers are defined using a DSL from the `clojurewerkz.quartzite.triggers` namespace. Lets define a trigger that fires
(executes its associated job) 10 times every 200 ms, starting immediately:

{% gist cfa1d575050843aeca35 %}

So how triggers are defined is quite similar to how jobs are defined. Quartz provides several types of execution schedules
and Quartzite supports all of them in the DSL. A couple more schedule types will be demonstrated later in this guide.

Simple periodic schedule demonstrated here is used when you need to perform a task N times with a fixed time interval.


## Using keys to identify triggers

In order to pause or completely remove a job from the scheduler, there needs to be a way to identify it. Job identifiers are called "keys". A key consists
of a string identifier and an (optional) group. To instantiate keys, use `clojurewerkz.quartzite.triggers/key` function:

{% gist 9bebc5795ef1bf696c39 %}

When group is not specified, the default group is used. It is common to use groups to "namespace" triggers, for example, to separate triggers
that schdule periodic data aggregation from those that generate invoices.


## Using Cron expression schedules

One of the schedule types that Quartz supports is the Cron expression schedule. It lets you define the schedule as a single expression used
by cron(8)(http://linux.die.net/man/8/cron). This form is concise but may also seem cryptic. As such, Cron schedules
are most commonly used when migrating legacy applications or by developers who are deeply familiar with Cron.

To define a trigger that will use a Cron expression schedule, you combine DSLs from `clojurewerkz.quartzite.triggers` and `clojurewerkz.quartzite.schedule.cron` namespaces:

{% gist 4fdad0672e53b96b5732 %}

To learn more about Cron expressions, consult [crontab(5)](http://linux.die.net/man/5/crontab).


## Using calendar interval schedules

Third type of trigger schdule is the calendar interval schedule. It fires at fixed intervals: minutes, hours, days, weeks, months or years.
In this example, we use intervals of 1 day:

{% gist aa70ce2fbf668794d4f8 %}


## Using daily interval schedules

Daily interval schedules make it easy to define schedules like

 * "Monday through Friday from 9 to 17"
 * "Every weekend at 3 in the morning"
 * "Every Friday at noon"
 * "Every day at 13:45"
 * "Every hour on Thursdays but not later than 15:00, up to 400 times total"

without having to deal with Cron expressions:

{% gist 0ec2b70b395d754d8409 %}


## Jobs contexts and passing data maps to jobs

TBD


## Pausing and resuming triggers

TBD


## Completely removing triggers from the scheduler

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
