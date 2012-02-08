# Leiningen Core

This library provides the core functionality of Leiningen. This
consists of the task execution implementation and helper functions
without any of the tasks or launcher scripts.

More [copious documentation is available](http://technomancy.github.com/leiningen/).

The tasks that get run come from Leiningen itself as well as any
Leiningen plugins that may be active.

## Namespaces

* **leiningen.core.main** contains the `-main` entry point along with
    task handling functions like `apply-task` and `resolve-task`.
* **leiningen.core.project** has `read` and `defproject` for getting a
    project map from `project.clj` files. It also handles applying
    profiles to the project map and loading plugins.
* **leiningen.core.classpath** is where the project's classpath is
    calculated. It handles Maven dependencies as well as checkout
    dependencies.
* **leiningen.core.eval** houses the `eval-in-project` function which
    implements the isolation of project code from Leiningen's own code.
* **leiningen.core.user** just has a handful of functions which handle
    user-level configuration.
* **leiningen.core.ns** contains helper functions for finding
    namespaces on the classpath.

## Running Tasks

When Leiningen is invoked, it first reads the `project.clj` file and
applies any active profiles to the resulting project map. (See
Leiningen's own readme for a description of how profiles work.) Then
it looks up the task which was invoked. Tasks are just functions named
after the task they implement and defined in the `leiningen.the-task`
namespace. They usually take a project map as their argument, but can
also run outside the context of a project. See the
[plugin guide](https://github.com/technomancy/leiningen/blob/stable/doc/PLUGINS.md)
for more details on how tasks are written. The `apply-task` function
looks up the task function, checks to make sure it can be applied to
the provided arguments, and then calls it.

## Project Isolation

When you launch Leiningen, it must start an instance of Clojure to
load itself. But this instance must not affect the project that you're
building. It may use a different version of Clojure or other
dependencies from Leiningen itself, and Leiningen's code should not be
visible to the project's functions.

Leiningen currently implements this by launching a sub-process using
`leiningen.core.eval/eval-in-project`. Any code that must execute
within the context of the project (AOT compilation, test runs, repls)
needs to go through this function. This sub-process (referred to as
the "project JVM") is an entirely new invocation of the `java` command
with its own classpath calculated from functions in the
`leiningen.core.classpath` namespace. It can only communicate with
Leiningen's process via the file system, sockets, and its exit code.

The exception to this rule is when `:eval-in-leiningen` in
`project.clj` is true, as is commonly used for Leiningen plugins.
Since Leiningen plugins are intended to be used inside Leiningen
itself, there's no need to enforce this isolation.

## License

Copyright © 2011-2012 Phil Hagelberg and 
[contributors](https://www.ohloh.net/p/leiningen/contributors). 

Distributed under the Eclipse Public License, the same as Clojure.