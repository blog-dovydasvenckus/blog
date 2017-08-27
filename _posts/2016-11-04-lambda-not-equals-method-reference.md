---
layout: post
title:  "Lambda != method reference"
description: Comparison between Java 8 method reference and lambda expression. One pitfall of method reference that you should avoid.
date:   2016-11-05 00:02:59 +0300
categories: java
---

## Introduction
Most of the IDE's give you hints, that you could replace lambdas with method references.
But there is a subtle difference between them.

I was refactoring commit at work on Vaadin application.
To improve readability I started replacing lambdas with method references.
After refactoring, and testing I noticed, that button click action resulted in NullPointerException.

## Example
In this example I'll try to show the difference between lambda expresion and method reference.

### Lambda
{% highlight java %}
package com.dovydasvenckus.lambda;

class MyCoolClass {
    public void doSomething() {
        System.out.println("My cool function invoked");
    }
}

public class Main {

    static MyCoolClass myCoolClass;

    public static void main(String[] args) {
        Runnable runnable = () -> myCoolClass.doSomething();
        myCoolClass = new MyCoolClass();
        runnable.run();
    }
}
{% endhighlight %}

#### Output:

    My cool function invoked

### Method reference

{% highlight java %}
Runnable runnable = myCoolClass::doSomething;
myCoolClass = new MyCoolClass();
runnable.run();
{% endhighlight %}

#### Output:

{% highlight java %}
Exception in thread "main" java.lang.NullPointerException
	at java.util.Objects.requireNonNull(Objects.java:203)
	at com.dovydasvenckus.lambda.Main.main(Main.java:14)
{% endhighlight %}

## Summary
Object must be initialized before using method reference operator on it.

Lambdas are bit different, they can access variable from outside their scope.
