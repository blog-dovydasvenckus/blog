---
layout: post
title:  "REST API using Jersey on embedded Jetty server"
description: Tutorial for setting up lightweight REST API using Jersey, Jetty and Gradle.
date:   2017-08-20 17:00:00 +0300
categories: rest
image: /assets/images/common/thumbnails/java.png
---

## Introduction
In this tutorial I'll show how to setup REST web service using Jersey on
embedded Jetty server. For build tool I'll be using [Gradle](https://gradle.org/).
Also, we will package up this application as fatjar, single executable Jar for easy deployment.

[Jersey](https://jersey.github.io/) is [JAX-RS](https://github.com/jax-rs) implementation.
This framework allows easy development of RESTful Web services.

Jetty is a lightweight HTTP server that can be easily embedded into the jar.

## Example project source code
You can find the code used in this example in github
[repository](https://github.com/blog-dovydasvenckus/jersey-with-embedded-jetty).

## Project structure
```bash
$ tree
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── settings.gradle
└── src
    └── main
        └── java
            └── com
                └── dovydasvenckus
                    └── jersey
                        ├── JerseyApplication.java
                        └── resources
                            └── HelloResource.java

```

This is a quite simple project structure. Most of the files are gradle config files
and gradle wrapper. We will be touching only 3 files: *build.gradle*, *HelloResource.java*,
*JerseyApplication.java*.

## Build config
This is how *build.gradle* file looks like:

```gradle
apply plugin: 'java'
apply plugin: 'application'
apply plugin: 'com.github.johnrengelman.shadow'

sourceCompatibility = 1.8
mainClassName = 'com.dovydasvenckus.jersey.JerseyApplication'

ext {
    slf4jVersion = '1.7.25'
    jettyVersion = '9.4.6.v20170531'
    jerseyVersion = '2.25.1'
}

buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath 'com.github.jengelman.gradle.plugins:shadow:2.0.1'
    }
}

repositories {
    jcenter()
}

dependencies {
    compile "org.slf4j:slf4j-api:${slf4jVersion}"
    compile "org.slf4j:slf4j-simple:${slf4jVersion}"

    compile "org.eclipse.jetty:jetty-server:${jettyVersion}"
    compile "org.eclipse.jetty:jetty-servlet:${jettyVersion}"

    compile "org.glassfish.jersey.core:jersey-server:${jerseyVersion}"
    compile "org.glassfish.jersey.containers:jersey-container-servlet-core:${jerseyVersion}"
    compile "org.glassfish.jersey.containers:jersey-container-jetty-http:${jerseyVersion}"
}
```

I think gradle file is pretty self explanatory. Probably only thing you should change
is mainClassName variable to point to your main class that will initialize Jetty server.

## Server configuration
In JerseyApplication I have configured Jetty and Jersey.

```java
package com.dovydasvenckus.jersey;

import org.eclipse.jetty.server.Server;
import org.eclipse.jetty.servlet.ServletContextHandler;
import org.eclipse.jetty.servlet.ServletHolder;
import org.glassfish.jersey.servlet.ServletContainer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.eclipse.jetty.servlet.ServletContextHandler.NO_SESSIONS;

public class JerseyApplication {

    private static final Logger logger = LoggerFactory.getLogger(JerseyApplication.class);

    public static void main(String[] args) {

        Server server = new Server(8080);

        ServletContextHandler servletContextHandler = new ServletContextHandler(NO_SESSIONS);

        servletContextHandler.setContextPath("/");
        server.setHandler(servletContextHandler);

        ServletHolder servletHolder = servletContextHandler.addServlet(ServletContainer.class, "/api/*");
        servletHolder.setInitOrder(0);
        servletHolder.setInitParameter(
                "jersey.config.server.provider.packages",
                "com.dovydasvenckus.jersey.resources"
        );

        try {
            server.start();
            server.join();
        } catch (Exception ex) {
            logger.error("Error occurred while starting Jetty", ex);
            System.exit(1);
        }

        finally {
            server.destroy();
        }
    }
}

```
    Server server = new Server(8080);

Sets server port to 8080.

    ServletContextHandler servletContextHandler = new ServletContextHandler(NO_SESSIONS);

To use jetty you must create ServletContextHandler. In this particular instance, we
created handler without sessions, because REST is stateless.

    servletContextHandler.setContextPath("/");

Sets application path to root.

    ServletHolder servletHolder = servletContextHandler.addServlet(ServletContainer.class, "/api/*");

This adds Servlet that will handle requests on /api/*. That means our Web services
will be accessible using /api/{resource} path.

    servletHolder.setInitParameter(
            "jersey.config.server.provider.packages",
            "com.dovydasvenckus.jersey.resources"
    );

This is important step. You must set package where rest resources are located.

## REST resource

```java
package com.dovydasvenckus.jersey;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/hello")
public class HelloController {

    @GET
    @Path("/{param}")
    @Produces(MediaType.TEXT_PLAIN)
    public String getMessage(@PathParam("param") String msg) {
        return "Hello " + msg + "\n";
    }
}

```

This resource is located /api/hello. And it takes single path param.

## Building
    ./gradlew build

It should build fatjar in build/libs directory. The jar should be named {application-name}-all.jar.
In this case jar was named jersey-with-embedded-jetty-all.jar. You can find the application name
in settings.gradle, under rootProject.name property.

## Running
Launch it like any normal executable jar.

    java -jar build/libs/jersey-with-embedded-jetty-all.jar

It should start successfully on port 8080.
```
[main] INFO org.eclipse.jetty.util.log - Logging initialized @51ms to org.eclipse.jetty.util.log.Slf4jLog
[main] INFO org.eclipse.jetty.server.Server - jetty-9.4.z-SNAPSHOT
[main] INFO org.eclipse.jetty.server.handler.ContextHandler - Started o.e.j.s.ServletContextHandler@f0c8a99{/,null,AVAILABLE}
[main] INFO org.eclipse.jetty.server.AbstractConnector - Started ServerConnector@3e27ba32{HTTP/1.1,[http/1.1]}{0.0.0.0:8080}
[main] INFO org.eclipse.jetty.server.Server - Started @467ms
```

## Testing web service
```bash
$ curl localhost:8080/api/hello/John
Hello John
```

Using a linux tool curl I have send GET request to **/api/hello/John**
and received plain text response that contains message "Hello John".

## Final thoughts
That's it. It is pretty straight forward to create a lightweight REST API using
Jersey and Jetty, without any heavy weight framework.

If you have any questions feel free to drop them in comment section below.
