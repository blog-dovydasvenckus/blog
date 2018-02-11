---
layout: post
title:  "Being lazy in Java with orElseGet"
description: Lazily working with Optional and avoiding pitfalls of orElse
date:   2017-12-30 23:59:59 +0300
categories: java
tags: java java8 tips
image: /assets/images/common/thumbnails/java.png
---

[Optionals](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html)
 are really cool Java 8 feature. But it works differently than I thought.
So I'll show how I have shot my own leg when working with `orElse`.

```java
public static void main(String[] args) {
        Client client = findClientInDatabase()
                .orElse(createAndSaveNewClient());
    }

    private static Optional<Client> findClientInDatabase() {
        System.out.println("Searching for client in database");
        return Optional.of(new Client());
    }

    private static Client createAndSaveNewClient() {
        System.out.println("Creating new client");
        return new Client();
    }
```

This code should find the client or it should create new client **only if a client does not exist**.

Output:
```
Searching for client in database
Creating new client
```

As you can see it works differently, then I have expected.
`orElse` clause is always evaluated and the result is calculated.

## orElseGet to the rescue
`orElseGet` takes supplier function instead of value. And this supplier function
is executed lazily only when Optional value is `empty`.

```java
public static void main(String[] args) {
    Client client = findClientInDatabase()
            .orElseGet(() -> createAndSaveNewClient());
}

private static Optional<Client> findClientInDatabase() {
    System.out.println("Searching for client in database");
    return Optional.of(new Client());
}

private static Client createAndSaveNewClient() {
    System.out.println("Creating new client");
    return new Client();
}
```

```
Searching for client in database
```


## Conclusion
It is really easy to introduce bug because of `orElse` value is always calculated.
This example was taken from a real-world situation. I have almost merged code which
persisted a new entity to the database, even when an entry with the same name has already existed.
Thankfully integration tests have caught it.

Use `orElse` mindfully, I would recommend using `orElse` only when value is
constant or when `orElse` clause won't have side effects. `orElseGet` solves
this issue by loading values lazily.
