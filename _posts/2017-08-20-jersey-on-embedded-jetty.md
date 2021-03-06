---
layout: post
title:  "Lightweight REST API using Jersey on embedded Jetty server"
description: Tutorial for setting up lightweight REST API using Jersey, Jetty and Gradle.
date:   2017-08-20 17:00:00 +0300
categories: rest
tags: java jersey jetty rest jackson
image: /assets/images/common/thumbnails/java.png
---

In this tutorial, I'll show how to setup REST web service using Jersey on
embedded Jetty server. For build tool, I'll be using [Gradle](https://gradle.org/).
Also, we will package up this application as FatJar, single executable Jar for easy deployment.

[Jersey](https://jersey.github.io/) is [JAX-RS](https://github.com/jax-rs) implementation.
This framework allows easy development of RESTful Web services.

Jetty is a lightweight HTTP server that can be easily embedded into the jar.

## Example project source code
You can find the code used in this example in GitHub
[repository](https://github.com/blog-dovydasvenckus/jersey-with-embedded-jetty).

## Project structure
```bash
$ tree
.
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
                        ├── greeting
                        │   └── Greeting.java
                        ├── JerseyApplication.java
                        └── resources
                            └── HelloResource.java

```

This is a quite simple project structure. Most of the files are Gradle config files
and Gradle wrapper. We will be touching only 4 files: *build.gradle*, *Greeting.java*, *HelloResource.java*,
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
    jerseyVersion = '2.27'
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
    compile "org.glassfish.jersey.media:jersey-media-json-jackson:${jerseyVersion}"
    compile "org.glassfish.jersey.inject:jersey-hk2:${jerseyVersion}"
}
```

I think Gradle file is pretty self-explanatory. Probably only thing you should change
is mainClassName variable to point to your main class that will initialize Jetty server.

JSON parsing is done by using popular [Jackson](https://github.com/FasterXML/jackson) library.
Including `jersey-media-json-jackson` package in classpath will take care of JSON marshaling and unmarshaling.

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

Sets application path to the root.

    ServletHolder servletHolder = servletContextHandler.addServlet(ServletContainer.class, "/api/*");

This adds Servlet that will handle requests on /api/*. That means our Web services
will be accessible using /api/{resource} path.

    servletHolder.setInitParameter(
            "jersey.config.server.provider.packages",
            "com.dovydasvenckus.jersey.resources"
    );

This is an important step. You must set package where rest resources are located.

## Greeting POJO
This simple [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object) will
be used as transfer class. It will be marshaled to JSON and unmarshaled from JSON.

```java
package com.dovydasvenckus.jersey.greeting;

public class Greeting {

    private String message;

    Greeting() {

    }

    public Greeting(String name) {
        this.message = getGreeting(name);
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String name) {
        this.message = name;
    }

    private String getGreeting(String name) {
        return "Hello " + name;
    }
}

```

You must create default no args constructors for classes that will be marshaled
and unmarshaled.

## REST resource

```java
package com.dovydasvenckus.jersey.resources;

import com.dovydasvenckus.jersey.greeting.Greeting;

import javax.ws.rs.*;
import javax.ws.rs.core.MediaType;

@Path("/hello")
public class HelloResource {

    @GET
    @Path("/{param}")
    @Produces(MediaType.APPLICATION_JSON)
    public Greeting hello(@PathParam("param") String name) {
        return new Greeting(name);
    }

    @POST
    @Produces(MediaType.TEXT_PLAIN)
    public String helloUsingJson(Greeting greeting) {
        return greeting.getMessage() + "\n";
    }
}
```

This resource is located `/api/hello`. There are two endpoints.
First uses GET method, takes string path param and returns JSON.
Second uses POST method, parses JSON and returns plain text.

## Building
    ./gradlew build

It should build FatJar in build/libs directory. The jar should be named {application-name}-all.jar.
In this case, jar was named jersey-with-embedded-jetty-all.jar. You can find the application name
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
### JSON unparsing
```bash
$ curl localhost:8080/api/hello/John
{"message":"Hello John"}
```

Using a Linux tool curl I have send GET request to `/api/hello/John`
and received JSON response that contains message "Hello John".

### JSON parsing
```bash
$ curl -H "Content-Type: application/json" -X POST -d '{"message": "Hello John"}' http://localhost:8080/api/hello
Hello John
```

Sending POST method with JSON body returns correct plain text response.

## Final thoughts
That's it. It is pretty straightforward to create a lightweight REST API using
Jersey and Jetty, without any heavyweight framework.

If you have any questions feel free to drop them in the comment section below.
