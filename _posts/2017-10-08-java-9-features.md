---
layout: post
title:  "Java 9 features"
description: Java 9 features
date:   2017-10-08 23:45:00 +0300
categories: java
---

## Java 9 features
Java 9 is a major release, which brings highly anticipated modularization.

I'm late to Java 9 feature release party. But I'll try to list all features that
have captured my attention.

### Modularization (Project Jigsaw)
Most notable feature in this release is Java module system a.k.a "Jigsaw".

Module system should solve two problems: encapsulation and dependency declaration.

Before Java 9 all public classes loaded from external Jars were accessible in
the project. Because of this reason it is easy to accidentally expose API's that
are meant to be used privately. The new module system addresses it by defining exposed
packages in `module-info.java` file.

New module system lets to describe dependencies in `module-info.java` file.

**budget module-info.java**:
```java
module com.dovydasvenckus.budget {
    exports com.dovydasvenckus.budget;

    requires com.dovydasvenckus.ledger;
}
```

**ledger module-info.java**:
```java
module com.dovydasvenckus.ledger {
    exports com.dovydasvenckus.ledger
}
```

In this example, we have defined budgeting module that exposes only classes from
`com.dovydasvenckus.budget` package. Exposed package children are not exposed,
for example classes in `com.dovydasvenckus.budget.helper` package won't be exposed
to modules that import it.

In this example, I have also defined dependency our budget module depends on ledger
module. Ledger module exposes only classes in `com.dovydasvenckus.ledger` package.

Also JDK was modularized with this release, using **jlink** tool you can create
minimal JRE, that will be deployed with your application. If your application
does not require java.sql module you can make JRE that won't include it.
It makes sense to deploy lightweight JRE on embedded devices or when we are using
microservices that do not require full JDK.
More on it [Openjdk docs](http://openjdk.java.net/projects/jigsaw/quick-start#linker).

This topic is too wide to describe in this blog post, so I highly recommend
to visit official documentation [OpenJDK Jigsaw quick start](http://openjdk.java.net/projects/jigsaw/quick-start).

### HTTP 2 client
Major browsers supported HTTP/2 for a long time, finally Java have caught up and
introduced HTTP/2 client. This new client also supports WebSockets.

This module is released in incubation state. You can use incubating modules now, but
they will be officially released in a future release.
In JDK10 it should be moved from `jdk.incubator.httpclient` to `java.httpclient` module.
Also JDK10 might introduce breaking changes.

### Java shell (Jshell)
Finally Java has introduced REPL(Read Evaluate Print Loop) shell that allows to
run Java commands interactively. Sometimes it is really usefull to test code snippet,
for this purpose I have used GroovyConsole. But GroovyConsole is not perfect, because
it uses Groovy language which is superset of Java.

```groovy
jshell> String hello = "Hello World"
hello ==> "Hello World"
jshell> System.out.println(hello)
Hello World
```

As you can see commands does not require semicolon to execute them.
Also shell suports **Tab** autocomplete, enjoy it :).

Also I had hope that shell would allow to execute commands from java file. But it
does not support this feature. In my opinion it would be nice to be able to write
Java script files and execute them using single command.
So for Java scripting I'll stick with Nashorn or Groovy.


### JavaDoc search
<img src="/assets/images/2017-10-08-java-9-features/java-doc-search.png">

This improvement is small, but convenient. Now Java docs support auto completable
search. Before this feature it was painfully slow to find something in Java API
documentation, because you had to navigate using mouse and you had to know
in which package class resides. Whenever I was searching for class
I have always googled, because it was faster way to navigate to class.

### Stream improvements
One of my favorite features of Java 8 was Streams. It changed the way many people write
code that deals with collections. Code written with streams is compact and easily
readable.

In Java 9 streams got even better.
Stream has 4 new methods:
* **dropWhile** discards first elements of the stream until condicion is met.
* **takeWhile** process items until condition is met.
* **iterate** lets write direct replacment for loops.
```java
Stream.iterate(0, i -> i < 100, i -> i + 1)
  .forEach(System.out::println);
```
* **OfNullable** Lets create stream without checking if element is null.
  When elemnt is non null it returns stream with single element, else it returns
  empty Stream.

### Easier creation of immutable Collections
Java 9 introduced static methods for different collections. Now it is easier to
create **immutable** lists, sets, maps.

```java
List<String> fruits = List.of("Apple", "Orange", "Banana");
Map<String, Integer> stock = Map.of("Apple", 7, "Orange", 20, "Banana", 15);
Set<Integer> numbers = Set.of(1, 2, 3, 4, 5);
```

### Optional improvements
In Java 9 release Optional type received some love.

Three new methods were added to Optional class.

* `void stream()` method transforms optional to stream
* `void ifPresentOrElse​(Consumer<? super T> action, Runnable emptyAction)``.
  Same as ifPresent, but includes method to execute when optional empty.
*	`Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)`.
It is similar to orElseGet, but instead of returning T object it returns
Optional<T>. Because of that you can chain multiple or() methods.

```java
Person nameLess = new Person("45122", null, null);

Optional<String> firstName = Optional.of(nameLess)
        .map(Person::getFirstName)
        .or(() -> localNameResolver.resolveName(nameLess.getPersonalId()))
        .or(() -> externalNameResolver.resolveName(nameLess.getPersonalId()));
```

If person have missing name, then it will call localNameResolver.resolveName().
If this method returns empty Optional, then it will call externalNameResolver.resolveName().


### Private methods in interfaces
Java 8 have introduced default method in interfaces. Java 9 added ability to add
private method to interfaces. So if two default methods use duplicated code,
you can move that code to private method.

```java
public interface HelloInterface {

    default String helloWorld() {
        return hello("World");
    }

    default String helloJohn() {
        return hello("John");
    }

    private static String hello(String name) {
        return String.format("Hello %s!", name);
    }
}
```

### Process API improvements
Java 9 has some improvements to processes API. It has added a few method to
deal with system processes.

For example, it was painful to get PID, you had to write boilerplate code that
was OS dependant. Now it is easily retrievable using a single command.

```java
ProcessHandle currentProcess = ProcessHandle.current();
System.out.println(currentProcess.getPid())
```

### Reactive Streams API
Java 9 has introduced Flow APIs. This will allow 3rd party vendors to implement
publish-subscribe frameworks. [JEP 266](http://openjdk.java.net/jeps/266) specification
contains 4 interfaces located in `java.util.concurrent.Flow` class.

These interfaces include:
* [Flow.Processor](http://download.java.net/java/jdk9/docs/api/java/util/concurrent/Flow.Processor.html)
* [Flow.Publisher](http://download.java.net/java/jdk9/docs/api/java/util/concurrent/Flow.Publisher.html)
* [Flow.Subscriber](http://download.java.net/java/jdk9/docs/api/java/util/concurrent/Flow.Subscriber.html)
* [Flow.Subscription](http://download.java.net/java/jdk9/docs/api/java/util/concurrent/Flow.Subscription.html)

### Minor changes that probably won't be noticed
* Now you can get current java version by using *java --version*, instead of
 *-version*. This operation will be compliant with unix philosophy,
 because verbal operations should have two dashes before and
 short hand operations should have single dash.
* Underscore character is reserved. You can't create variable `_` anymore.
  [JEP 213](http://openjdk.java.net/jeps/213)
* Stack walking API [JEP 259](http://openjdk.java.net/jeps/259)
* Applet API deprecation [JEP 289](http://openjdk.java.net/jeps/289)
* @Deprecated annotation has new two fields. **forRemoval** and **since**.
  [JEP 277](http://openjdk.java.net/jeps/277)
* Default garbage collector was changed to G1. [JEP 248](http://openjdk.java.net/jeps/248)
* Compact Strings [JEP 254](http://openjdk.java.net/jeps/254)
* Java Platform Logging API [JEP 264](http://openjdk.java.net/jeps/264)
* Try with resource improvements [JEP 213](http://openjdk.java.net/jeps/213)

## Conclusion
I have mentioned most notable features. There are probably plenty features
that I have missed. For me Java 9 release is quite an exciting one.

 Modularization is a hudge feature and in my opinion, it will take quite some time
 to it being widely adopted.

 Besides modularization Java 9 includes many small handy features. I'll try to
 adopt these new features in my code.
