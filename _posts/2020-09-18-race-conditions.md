---
title: "Hacking Banks With Race Conditions"
categories:
  - Hacking
---

How race conditions compromise the security of financial sites.

One of the biggest threats to modern online banking is a type of vulnerability called race conditions.

These vulnerabilities stem from simple programming mistakes that developers commonly make. And these mistakes have proved costly: race conditions have been used by hackers to steal money from online banks, stock brokerages, and cryptocurrency exchanges. (They have also been used by hackers to drink unlimited coffee at Starbucks, as you'll soon see in this post!)

Today, let's talk about how and why these vulnerabilities happen, how attackers have exploited it, and how to prevent it in your own applications.

## Crash Course Concurrency

In computer science, concurrency is the ability for different parts of a program to be executed simultaneously without affecting the final outcome of the computation.

When a program is developed with concurrency in mind, it can take full advantage of modern multithreading and multiprocessing systems, and thus drastically improve its performance.

Multithreading is the ability of a CPU to provide multiple threads of execution. These threads don't all execute at the same time but take turns using the CPU's computational power. With multithreading, other threads can continue taking advantage of the unused computing resources while they are idle. (For example, when the original thread is suspended while waiting for input or output to complete.)

While multiprocessing refers to the usage of multiple CPUs for computation at the same time.

Arranging the sequence of execution of multiple threads is called scheduling. There are many different scheduling algorithms in use, each optimized for different performance priorities.

## What is a Race Condition?

In computing, a race condition happens when two sections of code that are designed to be executed in a sequence were executed out of sequence.

Since the scheduling algorithm can swap between the execution of two threads at any time, you can't predict the sequence in which the threads execute each action.

For example, let's say that two concurrent threads of execution are each trying to increase the value of a global variable by 1. So in the end, the global variable will have the value of 2. (The example is taken from the Wikipedia page: <https://en.wikipedia.org/wiki/Race_condition>)

Ideally, the threads would be executed as such:

![](https://vkili.github.io/blog/assets/images/hacking-01.png)

But, if the two threads are run simultaneously, without resource locks (a mechanism that blocks other threads from operating on the same resource) or synchronization (a mechanism that ensures that threads that utilize the same resources so not execute simultaneously), the execution could be scheduled like this:

![](https://vkili.github.io/blog/assets/images/hacking-02.png)

In this case, the final value of the global variable becomes 1, which is incorrect. (The expected value is 2.)

This happens because the outcome of the execution of one thread depends on the outcome of another thread. When the two threads are executed simultaneously, unexpected outcomes can occur.

## What is a Race Condition Vulnerability?

A race condition becomes a vulnerability when it affects a security control mechanism. Hackers can then induce a situation in which a sensitive action is executed before a security control is complete. For this reason, race condition vulnerabilities are also referred to as Time of Check/Time of Use vulnerabilities.

Imagine if the two threads of the above example are executing something a little more sensitive: the transferring of money between bank accounts. The application would have to perform three subtasks to transfer the money correctly:

1.  Check if account A has enough balance.
2.  Add the money to account B.
3.  Deduct the money from account A.

Let's say that you own two bank accounts, account A and account B. You have $500 in account A and $0 in account B. Now, you initiate two money transfers of $500 from account A to account B.

Ideally, when two money transfer requests were initiated, the program should behave like this.

![](https://vkili.github.io/blog/assets/images/hacking-03.png)

But if you can send the two request simultaneously, you might be able to induce a situation in which the execution of the threads become like this:

![](https://vkili.github.io/blog/assets/images/hacking-04.png)

Note that in this scenario, you end up with more money than you started with! You essentially made an additional $500 appear out of thin air by exploiting a race condition vulnerability.

## Case Study: Unlimited Starbucks Coffee

A cool example of how race conditions have been exploited in real life was the Starbucks hack by security researcher Egor Homakov.

By exploiting a race condition on the gift card page, he found a way to generate an unlimited amount of credit on Starbucks gift cards for free.

This vulnerability lies in the functionality that transfers the gift card balance from one card to another. The website uses the following POST request to transfer credit:

```
POST /step1?amount=5&from=card_A&to=card_B
```

When this request is sent multiple times within a small time frame, the hacker can induce a situation in which he can transfer more money into card B than the account balance that he has on card A. This means that when executed for many times, this attack generates unlimited amounts of money on card B for free.

The root cause of this attack is a race condition between these three actions:

-   Check that card A has enough money to be transferred.
-   Transfer that amount to card B.
-   Deduct the balance of card A.

This is the breakdown of the race condition that caused the vulnerability:

![](https://vkili.github.io/blog/assets/images/hacking-05.png)

You can read more about the fun details of the exploit here.

[Hacking Starbucks for unlimited coffee](https://sakurity.com/blog/2015/05/21/starbucks.html)

## High-Risk Applications

Race conditions are used as a way to subvert access controls. So in theory, any application that has sensitive actions that rely on access control mechanisms could be vulnerable.

Most of the time though, hackers target race conditions on the websites of financial institutions. Because if a race condition could be found on a critical functionality like cash withdrawal, fund transfer or credit card payment, it could lead to infinite financial gains for the hacker.

Other high-risk applications include e-commerce sites, online games, and online voting systems.

Additionally, if an exploitable race condition was ever found on a site, it is possible that the site does not follow secure programming practices and thus the likelihood of it being vulnerable again is high.

## Exploiting Race Conditions

Most of the time, you can test for and exploit race conditions in web apps by sending multiple requests to the server simultaneously.

```
Bank Account balance: 3000
```

For example, if you want to see if you can withdraw more money than you have in your bank account, you can simultaneously send multiple requests for withdrawal to the server via the curl command.

```
curl (withdraw 3000) & (withdraw 3000) & (withdraw 3000) & (withdraw 3000) & (withdraw 3000) & (withdraw 3000)
```

Note that whether your attack succeeds or not would depend on the process scheduling algorithm of the server and is a matter of luck. However, the more requests you send within a short time frame, the better the odds that your attack will succeed.

## Preventing Race Conditions

The key to preventing race conditions is to implement safe concurrency. And the best way to do this is by using resource locks. Most programming languages that have concurrency abilities will also have some sort of locking functionality built-in. Please refer to the documentation of your chosen language to learn how to do it!

Beyond that, following secure coding practices (like the least privilege principle) and auditing code on a regular basis will reduce the likelihood of your application being hacked!
