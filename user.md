User documentation
==================

# 1 Code repositories

Source code is available online on GitHub service at the following address: https://github.com/akka-tracing-tool. At the time of creating this part of the documentation  Akka-Tracing-Tool consists of 6 repositories:

* akka-tracing-core - the core part of the Akka Tracing library,
* akka-tracing-docs - the documentation of the Akka Tracing library,
* akka-tracing-sbt - SBT plugin for Akka Tracing Tool,
* akka-tracing-examples - examples of usage of the Akka Tracing library,
* akka-tracing-tutorial - simple project that explains how to use Akka Tracing Tool,
akka-tracing-visualization - visualization tool for visualizing the traces.

Main functionality, i.e. tracing is provided by the **core part** and the **SBT plugin**. Remaining repositories provide showcase of usage and extensions.

# 2 Requirements

In order to meet all library requirements, it’s necessary to provide few more dependencies into the project. Important goal of this tool was to minimize required configuration and make it user friendly.

## 2.1 Database driver

One of the most important is driver to database: it was impossible to equip library with all possible ones that would be in use. Instead developer has to provide the driver that meets his requirements best.

Under the hood we’re relying on *Slick*, which is database query and access library from *Typesafe*. There is only one restriction: your database engine must be conform with it.

Probably the most useful database engine for project that doesn’t involve complex persistence layer is *SQLite*. According to SQLite’s website [1], it’s *“a software library that implements a self-contained, serverless, zero-configuration, transactional SQL database engine.”* Drivers to use that database engine are widely available in public repositories and it’s very easy to integrate them into project.

## 2.2 Akka Remote

Next important requirement that enables collecting messages is library Akka Remote. It permits to communicate between different actor systems. When project actually includes this dependency, no additional configuration is needed. Otherwise, a simple configuration is necessary to introduce. However, in most cases example configuration file available on *akka-tracing-examples* is enough.

## 2.3 AspectJ

Our library use *runtime-weaving* to instrument actors. Hence to make tracing possible, it’s essential to introduce artifacts connected with AspectJ library: *Runtime* and *Weaver*. They are also widely available in public repositories. Also, our plugin includes them into project automatically and provide necessary options for JVM so you don’t need to bother it.
