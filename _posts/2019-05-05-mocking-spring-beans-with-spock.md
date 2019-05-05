---
layout: post
title:  "Mocking Spring beans with Spock"
description: Mocking Spring beans with Spock.
date:   2019-05-05 19:00:00 +0300
categories: java
tags: java spring groovy spock
image: /assets/images/common/thumbnails/spring.png
twitter_card.image: /assets/images/common/thumbnails/spring.png
---

I love Spock testing framework, it has much more elegant syntax that JUnit.
Also, I think it has a gentle learning curve compared to JUnit.
Not so long ago to Mock or Stub Spring context beans was a pain in the butt.
But Spock 1.2 version introduced an easier way to Mock and Stub spring beans.

In my examples I'll be using Spring Boot 2.1.

## Old way
In the old days to Mock or Stub Spring bean, we had to create separate Spring configuration which
created our beans using `DetachedMockFactory`.

**Example of old way:**

```groovy
@SpringBootTest
@Import(TestConfig)
class AccountServiceIntegrationSpec extends Specification {

    @Autowired
    AccountRepository accountRepository

    @Autowired
    AccountService accountService

    def 'should save new account'() {
        given:
            AccountDTO account = new AccountDTO(name: "Income", type: INCOME)
            Account accountWithId = new Account(id: 5, name: "Income", type: INCOME)
            accountRepository.save(_ as Account) >> accountWithId

        when:
            AccountDTO result = accountService.createAccount(account)

        then:
            with(result) {
                id == 5
                name == 'Income'
                type == INCOME
            }
    }

    static class TestConfig {
        DetachedMockFactory mockFactory = new DetachedMockFactory()

        @Bean
        @Primary
        AccountRepository accountRepositoryStub() {
            return mockFactory.Stub(AccountRepository)
        }
    }
}
```

As you can see it contains quite a lot boilerplate code to Stub single bean. And it was error prone, you had to remember to `@Import` your `TestConfig` class.

## Post Spock 1.2 way
Thankfully Spock 1.2 improved bean Mocking and Stubbing quite a lot and it became a trivial operation.

**Example:**
```groovy
@SpringBootTest
class AccountServiceIntegrationSpec extends Specification {

    @SpringBean
    AccountRepository accountRepository = Stub()

    @Autowired
    AccountService accountService

    def 'should save new account'() {
        given:
            AccountDTO account = new AccountDTO(name: "Income", type: INCOME)
            Account accountWithId = new Account(id: 5, name: "Income", type: INCOME)
            accountRepository.save(_ as Account) >> accountWithId

        when:
            AccountDTO result = accountService.createAccount(account)

        then:
            with(result) {
                id == 5
                name == 'Income'
                type == INCOME
            }
    }
}
```

Everything is done with only two lines. It differs from regular Stubbing only by single `@SpringBean` annotation.
```groovy
    @SpringBean
    AccountRepository accountRepository = Stub()
```

To use this `@SpringBean` annotation you must include [`org.spockframework:spock-spring`](https://mvnrepository.com/artifact/org.spockframework/spock-spring) dependency in your `classpath`.

Also, there is `@SpringSpy` annotation for creating spies, but generally I'm against spying, because usage of Spies, in my opinion, is bad code smell.

## Summary

Spock 1.2 version solved an old issue with Mocking and Stubbing Spring beans. You can read more about the latest Spock releases on their [GitHub page](https://github.com/spockframework/spock/blob/master/docs/release_notes.adoc).

If you have any questions feel free to leave a comment below.

Happy testing :)
