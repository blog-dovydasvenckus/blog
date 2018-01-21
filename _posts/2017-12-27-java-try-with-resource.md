---
layout: post
title:  "In-depth look at Java try-with-resource"
description: Java try-with-resource. How to implement resources that will be closed automatically.
date:   2017-12-27 23:45:00 +0300
categories: java
image: /assets/images/common/thumbnails/java.png
---

## Resource handling
Before Java 7 working with IO resource (files, JDBC) was tedious, you had
to close the resource manually. Most of these operations with resources can throw
exceptions. Whether the exception was thrown or try block completed without exception
resource should be closed.

### Pre Java 7 example:
Before Java 7 resource closing was mostly done using [finally](https://docs.oracle.com/javase/tutorial/essential/exceptions/finally.html).

{% highlight java %}
public static void main(String[] args) throws IOException {
     BufferedWriter writer = new BufferedWriter(new FileWriter("test.txt"));

     try {
         writer.write("My test text");
     } catch (IOException ioException) {
         // Handle exception
     } finally {
         writer.close();
     }
 }
{% endhighlight %}

### Post Java 7 example:
{% highlight java %}
public static void main(String[] args) {
    try (BufferedWriter writer = new BufferedWriter(new FileWriter("test.txt"))) {
        writer.write("My test text");
    } catch (IOException ioException) {
        System.out.println("Handling exception");
    }
}
{% endhighlight %}


As you can see Java 7 sample is a lot shorter, it lacks finally clause.

In Java 7 resource closing can be delegated to the resource. For this to work
you must initialize resource after **try** clause, in parenthesis. Regardless
if an exception was thrown or try block completed without exception, close method is called.

## Implementing your own auto closable resource
At first, I thought that this feature was really cool.
My second thought was "What kind of sorcery is this". Under the hood, it is really simple,
resource must implement [AutoCloseable](https://docs.oracle.com/javase/9/docs/api/java/lang/AutoCloseable.html) interface.
This interface has only single method `void close​() throws Exception`.

**MyFirstAutoCloseableResource.java**:
{% highlight java %}
package com.dovydasvenckus.resource;

public class MyFirstAutoCloseableResource implements AutoCloseable {

    public void doSomething() {
        System.out.println(getClassName() + ": doing stuff");
    }

    @Override
    public void close() throws Exception {
        System.out.println(getClassName() + ": closing resource");
    }

    private String getClassName() {
        return this.getClass().getSimpleName();
    }
}
{% endhighlight %}

**Main.java**:
{% highlight java %}
package com.dovydasvenckus.resource;

public class Main {
    public static void main(String[] args) {
        try (MyFirstAutoCloseableResource first = new MyFirstAutoCloseableResource()) {
            first.doSomething();
        }
        catch (Exception ex) {
            System.out.println("Handling exception");
        }
    }
}
{% endhighlight %}

Executing this code should print:
```
MyFirstAutoCloseableResource: doing stuff
MyFirstAutoCloseableResource: closing resource
```

To demonstrate execution order when an exception occurs I have updated **doSomething** method.

{% highlight java %}
public void doSomething() {
        System.out.println(this.getClass().getSimpleName() + ": doing stuff");
        throw new RuntimeException("Some exception");
    }
{% endhighlight %}

```
MyFirstAutoCloseableResource: doing stuff
MyFirstAutoCloseableResource: closing resource
Handling exception
```
As you can see resource was closed before handling the exception.

**Tip!** While implementing close method you should specify a more specific type
of Exception. Also, you can specify RuntimeException, this makes **catch** block
optional.

## Exception propagation differences

### Finally resource closing
When using old style of resource closing **catch block** exceptions are swallowed,
when the exception occurs in **finally** block and only exception from finally is thrown.

{% highlight java %}
public static void main(String[] args) {
        try {
            methodThatThrowsException();
        }

        catch (RuntimeException exception) {
            System.out.println(exception);
        }
        return;
    }

    private static void methodThatThrowsException() {
        try {
            System.out.println("Throwing exception in try block");
            throw new RuntimeException("Try block exception.");
        } finally {
            System.out.println("Throwing exception in finally block");
            throw new RuntimeException("Finally block exception");
        }
    }
{% endhighlight %}

**Output**:
```
Throwing exception in try block
Throwing exception in finally block
java.lang.RuntimeException: Finally block exception
```

### try-with-resource
**FirstAutoCloseableResource.java**
{% highlight java %}
package com.dovydasvenckus.resource;

public class FirstAutoCloseableResource implements AutoCloseable {

    public void doSomething() {
        System.out.println("Throwing exception from doSomething()");
        throw new RuntimeException("DoSomething() exception");
    }

    @Override
    public void close() throws RuntimeException {
        System.out.println("Throwing exception from close()");
        throw new RuntimeException("close() exception");
    }
}
{% endhighlight %}

**Main.java**
{% highlight java %}
package com.dovydasvenckus.resource;

public class Main {
    public static void main(String[] args) {
        try (FirstAutoCloseableResource first = new FirstAutoCloseableResource()) {
            first.doSomething();
        }
        catch (RuntimeException ex) {
            System.out.println(ex);
        }
    }
}
{% endhighlight %}

**Output**:
```
Throwing exception from doSomething()
Throwing exception from close()
java.lang.RuntimeException: DoSomething() exception
```

In this case exception from action was propagated instead of close method exception.
Using try-with-resource you can retrieve exception from close() method by
calling **getSuppressed()** on doSomething() exception.

## Multiple resources
You can define multiple resources in try statement, Java will handle closing for you.

**FirstAutoCloseableResource.java**:
{% highlight java %}
package com.dovydasvenckus.resource;

public class FirstAutoCloseableResource implements AutoCloseable {

    public void doSomething() {
        printWithClassName("Executing doSomething()");
    }

    @Override
    public void close() throws RuntimeException {
        printWithClassName("Closing resource");
    }

    private void printWithClassName(String text) {
        System.out.println(this.getClass().getSimpleName() + ": " + text);
    }
}
{% endhighlight %}

**SecondAutoCloseableResource.java**:
{% highlight java %}
package com.dovydasvenckus;

public class SecondAutoCloseableResource extends FirstAutoCloseableResource {
}
{% endhighlight %}

**MultipleResources.java**:
{% highlight java %}
package com.dovydasvenckus.resource;

public class MultipleResources {

    public static void main(String[] args) {
        try (
                FirstAutoCloseableResource resource1 = new FirstAutoCloseableResource();
                SecondAutoCloseableResource resource2 = new SecondAutoCloseableResource()
        ) {
            resource1.doSomething();
            resource2.doSomething();
        }
    }
}
{% endhighlight %}

**Output**:
```
FirstAutoCloseableResource: Executing doSomething()
SecondAutoCloseableResource: Executing doSomething()
SecondAutoCloseableResource: Closing resource
FirstAutoCloseableResource: Closing resource
```

Also, you should keep in mind that Java closes resources in backward declaration order.
It will start closing resources from last to first.

## Java 9 additions
Before Java 9 you could not initialize **AutoCloseable** resource outside try statement.
Java 9 allows us to define auto closable resources outside try scope.

**ResourceOutsideTry.java**:
{% highlight java %}
package com.dovydasvenckus.resource;

public class ResourceOutsideTry {

    public static void main(String[] args) {
        FirstAutoCloseableResource resource = new FirstAutoCloseableResource();

        try (resource) {
            resource.doSomething();
        }
    }
}
{% endhighlight %}

This might be useful in cases when the resource was passed to function or using dependency
injection. But this has one downside, after cleanup you can still access the resource
and invoke methods on it, this could lead to undesirable behavior.

## Conclusion
Try-with-resource reduces boilerplate code and helps to avoid human error in case
we forget to close resource.
I hope you learned something from this long-winded blog post about this simple feature.

If you have any questions feel free to leave a comment.
