---
title: "Using durable Quartz stores with Quartzite"
layout: article
---

## About this guide

This guide covers:

 * Overview of durable and transients job stores
 * Available durable job stores
 * Quartz and Clojure class loaders
 * How to use durable job stores with Quartzite

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/quartzite.docs).


## Overview

During Quartz operation, it has to do some housekeeping: track trigger execution history,
trigger and job state, misfires and so on. By default Quartz keeps this information
in memory. In some cases it is all you need but has obvious drawbacks:

 * If JVM is stopped, killed or crashes, all state is lost
 * Really large number of jobs and triggers may consume a lot of RAM

To address this issue, Quartz supports *durable job stores*. They
store all the scheduler state, not just information about jobs.


## Available Durable Quartz Job Stores

Quartz comes with a JDBC-backed job data store out of the box. There is also a [MongoDB-backed one](https://github.com/michaelklishin/quartz-mongodb).

For more information, see Quartz documentation on [JDBC-backed durable Quartz stores](http://quartz-scheduler.org/documentation/quartz-2.x/configuration/ConfigJobStoreTX).


## Quartz and Clojure Class Loaders

Clojure is a compiled language: code is compiled when it is loaded (usually via `clojure.core/require`). To dynamically
load classes Clojure code compiles to, Clojure uses a special class loader. Since Quartz also uses its own class loader,
due to the JVM security model it cannot see job classes Clojure compiler generates.

A solution to this issue is described in the following section.


## How to use durable job stores with Quartzite

A solution to the different class laoders issue is to subclass the job store class and (if it allows this) make it use
Clojure's `clojure.lang.DynamicClassLoader`:

``` java
package megacorp.myservice.quartz;

import clojure.lang.DynamicClassLoader;
import com.novemberain.quartz.mongodb.MongoDBJobStore;

public class JobStore extends MongoDBJobStore implements org.quartz.spi.JobStore {
  @Override
  protected ClassLoader getJobClassLoader() {
    // makes it possible for Quartz to load and instantiate jobs that are defined
    // using defrecord without AOT compilation.
    return new DynamicClassLoader();
  }
}
```

and then configure Quartz to use it via the `quartz.properties` file:

```
org.quartz.scheduler.instanceName = MyServiceScheduler
org.quartz.threadPool.threadCount = 4

## Use MongoDB-backed store to persistently store information about
# scheduled jobs, triggers and their state.
#
org.quartz.jobStore.class=megacorp.myservice.quartz.JobStore
org.quartz.jobStore.addresses=127.0.0.1
org.quartz.jobStore.dbName=myservice_production
org.quartz.jobStore.collectionPrefix=quartz

## Quartz plugins
#
org.quartz.plugin.triggHistory.class = org.quartz.plugins.history.LoggingTriggerHistoryPlugin
org.quartz.plugin.jobHistory.class = org.quartz.plugins.history.LoggingJobHistoryPlugin
```

For more information about mixed Clojure/Java projects, see [Clojure/Java projects with Leiningen 2+](https://github.com/technomancy/leiningen/blob/master/doc/MIXED_PROJECTS.md).

Future versions of Quartzite may ship with custom job classes like this out of the box.


## Wrapping Up

Quartz provides support for durable stores for its state. Using a durable store
is a good idea for availability reasons. Due to specifics of how Clojure compiler
and Quartz scheduler work, using durable stores with Quartzite requires a little
bit of glue code. Quartzite authors continue looking for a good generic solution.


## What version of Quartzite does this guide cover?

This guide covers Quartzite `1.1.x` (including beta releases).
