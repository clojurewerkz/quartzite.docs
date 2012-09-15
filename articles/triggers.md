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

This guide covers Quartzite `1.0.x`.


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

Triggers can be paused and resumed. Paused triggers will not fire and associated jobs will not be executed.
To pause a trigger, use `clojurewerkz.quartzite.scheduler/pause-trigger` and pass it the trigger's key:

{% gist 64c07a8476c07114ffc4 %}

`clojurewerkz.quartzite.scheduler/pause-triggers` will pause one or several groups of triggers. What groups
are paused is determined by the *group matcher*, instantiated via Java interop:

{% gist 553404903de55b0ee524 %}

In addition to the exact matcher, there are several other matchers available:

{% gist dcd5d0191fa4dea461d3 %}

Resuming a trigger makes it fire again. `clojurewerkz.quartzite.scheduler/resume-trigger` is the function that does
that for a single trigger:

{% gist f170d475f4c1b684fe24 %}

`clojurewerkz.quartzite.scheduler/resume-triggers` resumes one or more trigger groups using the already covered group matchers:

{% gist 55b59aab0cdee57192d4 %}

Finally, `clojurewerkz.quartzite.scheduler/pause-all!` and `clojurewerkz.quartzite.scheduler/resume-all!` are functions
that pause and resume *the entire scheduler*. Use them carefully. Both take no arguments.


## Completely removing trigger from the scheduler

It is possible to completely remove a trigger from the scheduler. Doing so will also remove *all the associated triggers*. The trigger
will never be executed again (unless it is re-scheduled). `clojurewerkz.quartzite.scheduler/delete-trigger` deletes
a single trigger by trigger key:

{% gist d810569ddd29ef14aa38 %}

while `clojurewerkz.quartzite.scheduler/delete-triggers` removes multiple triggers and takes a collection of keys:

{% gist aaf691775e539d0e1949 %}



## Misfires

A misfire occurs if a persistent trigger "misses" its firing time because of being paused, the scheduler being shutdown, or because there are no
available threads in Quartz's thread pool for executing the job.

The different trigger types have different misfire instructions available to them. By default they use a 'smart policy' instruction which has dynamic
behavior based on trigger type and configuration. When the scheduler starts, it searches for any persistent triggers that have misfired, and it then
updates each of them based on their individually configured misfire instructions.

Not all misfire instructions make sense for all triggers. Different triggers use different schedules and thus misfire instructions
available to them depend 

### Misfire Instructions Available to All Triggers

#### The "Ignore Misfires" Instruction

The "ignore misfires" instruction instructs the scheduler should simply fire the trigger as soon as it can, as many
times as necessary to catch back up with the schedule. This means the trigger may fire multiple times in rapid succession.

A code example that uses this instruction:

{% gist 31a9fa1932ce837716cf %}



### Misfire Instructions Available to Triggers That Use the Simple Schedule

#### The "Fire Now" Instruction

Instructs the scheduler to fire the trigger immediately upon misfire. Should only be used for one-shot
(non-repeating) timers.

A code example that uses this instruction:

{% gist 3f0a3fb21d10138161e9 %}

#### The "Next With Existing Repeat Count" Instruction

Instructs the scheduler to re-schedule the trigger to the next scheduled time after "now" with
the repeat counter unchanged. If the end time of the trigger has arrived, the trigger will be
marked as completed (will not fire again).

A code example that uses this instruction:

{% gist dd72364b8f74af19fd93 %}

#### The "Next With Remaining Repeat Count" Instruction

Instructs the scheduler to re-schedule the trigger to the next scheduled time after "now"
and changes the repeat counter to what it would be, if it had not missed any fires.

If the end time of the trigger has arrived, the trigger will be marked as completed (will not fire again).

A code example that uses this instruction:

{% gist ee796fd54ee3cfa30581 %}

#### The "Now With Existing Repeat Count" Instruction

Instructs the scheduler to fire the trigger "now" and not change the repeat count. If "now" is after the trigger's
end time, the trigger will not fire.

A code example that uses this instruction:

{% gist 465328e3ddc308348f1b %}

#### The "Now With Remaining Repeat Count" Instruction

Instructs the scheduler to fire the trigger "now" and changes the repeat counter to what it would be, if it had not
missed any fires. If "now" is after the trigger's end time, the trigger will not fire.

A code example that uses this instruction:

{% gist 3594aca7d799cb6403ba %}



### Misfire Instructions Available to Triggers That Use the Daily Time Interval Schedule

#### The "Fire Once Now" Instruction

Instructs the scheduler to fire the trigger immediately upon misfire.

A code example that uses this instruction:

{% gist 9a150703f13205abb3d2 %}

#### The "Do Nothing" Instruction

Instructs the scheduler to do nothing.

A code example that uses this instruction:

{% gist 5376047d4930d1ff1647 %}



### Misfire Instructions Available to Triggers That Use the Calendar Interval Schedule

#### The "Fire Once Now" Instruction

Instructs the scheduler to fire the trigger immediately upon misfire.

A code example that uses this instruction:

{% gist c252862174dec3ce5eee %}

#### The "Do Nothing" Instruction

Instructs the scheduler to do nothing.

A code example that uses this instruction:

{% gist e88f2032091373409d8f %}



### Misfire Instructions Available to Triggers That Use the Cron Schedule

#### The "Fire Once Now" Instruction

Instructs the scheduler to fire the trigger immediately upon misfire.

A code example that uses this instruction:

{% gist 90f53d0fc2a0a64e21a5 %}

#### The "Do Nothing" Instruction

Instructs the scheduler to do nothing.

A code example that uses this instruction:

{% gist baf719b27a62118b056e %}




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
