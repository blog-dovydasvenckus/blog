---
layout: post
title:  "Failed assertThat throwing NoSuchMethodError exception when using Hamcrest and Mockito"
description: How to resolve java.lang.NoSuchMethodError exception, that occurs when using JUnit with Hamcrest and Mockito.
date:   2017-09-24 17:40:00 +0300
categories: junit
image: /assets/images/common/thumbnails/junit-logo.png
---

From the start of my career, I have preferred Spock for writing tests.
While working with legacy systems I have written few JUnit tests here and there.
But my knowledge JUnit, Hamcrest, and Mockito is fairly limited.

At work, I had a task that involved teaching intern how to write unit tests using JUnit.

After setting up Gradle project I have noticed, that failed assertThat statements
 threw **java.lang.NoSuchMethodError** exception with cryptic message.

    java.lang.NoSuchMethodError: org.hamcrest.Matcher.describeMismatch(Ljava/lang/Object;Lorg/hamcrest/Description;)V

## Example
It was gradle project with just 3 test dependencies.

build.gradle:

```gradle
apply plugin: 'java'

repositories {
    jcenter()
}

dependencies {
    testCompile 'junit:junit:4.12'
    testCompile 'org.mockito:mockito-all:1.10.19'
    testCompile 'org.hamcrest:java-hamcrest:2.0.0.0'
}
```

Test example:
```java
package com.dovydasvenckus.hamcrest;

import org.junit.Test;

import static org.hamcrest.Matchers.is;
import static org.junit.Assert.assertThat;

public class HamcrestMessageTest {

    @Test
    public void twoEqualsThree() {
        assertThat(2, is(3));
    }
}
```

Failed assertThat threw this cryptic exception.

## Solution

It fails because Mockito 1 depends on Hamcrest and JUnit has Hamcrest as a transient dependency.
So Mockito library starts using Hamcrest from JUnit. This causes described failure.

### Solution 1. You should be using Mockito 2
In this example, I have imported Mockito 1 version, because from Maven
repository I have accidentally selected mockito-all.jar. You should use mockito-core
jar to get Mockito 2. Mockito 2 does not depend on Hamcrest so this issue is resolved simply
by upgrading to Mockito 2.

    testCompile 'org.mockito:mockito-core:2.10.0'

### Solution 2. Importing Hamcrest before Mockito.
If you work on legacy project and you can't simply uplift the version, just import
Hamcrest before Mockito in build.gradle script.

Updated dependencies:
```gradle
dependencies {
    testCompile 'junit:junit:4.12'
    testCompile 'org.hamcrest:java-hamcrest:2.0.0.0'
    testCompile 'org.mockito:mockito-all:1.10.19'
}
```

### Result
```
java.lang.AssertionError:
Expected: is <3>
     but: was <2>
```

Failed assert messages are solved by both of the solutions.

## Final thoughts
Both of these solutions fixes the problem.

Even if you're working with legacy project I highly recommend upgrading to
the latest testing framework version.
There are some breaking changes, but if you won't upgrade now, the migration will
be harder in future, when the new framework version is released.

My coworker has migrated quite a large project to Hamcrest 2.0 and Mockito 2.1,
that is actively developed. He managed to upgrade Mockito and Hamcrest versions
in a short period of time. There were more than 500 test files that had to be updated
because some functionality was removed or changed. This migration turned out fine,
without any major problems.

There are few things that I have learned from this migration:
* Before merging such big pull request I recommend to inform developer community,
so it won't be a big surprise for them.
* When undertaking such task be prepared for merge conflicts. You might need to
  merge your pull request on weekend, if development is too active on working days.
* Also, you might piss off some developers, that have OPEN pull requests,
  because they might get merge conflicts after your changes were merged to master.
  But I wouldn't worry too much about that because you have done positive change to project.

If you have any questions or insights feel free to leave a comment.
