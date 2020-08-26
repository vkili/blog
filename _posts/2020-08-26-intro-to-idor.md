---
title: "Intro to IDORs"
categories:
  - Info Leak
---

And how to cause a massive data breach.

![](https://vkili.github.io/blog/assets/images/idor-01.png)

*Hey: Please remember that trying this on systems where you don't have permission to test is illegal. If you've found a vulnerability, please disclose it responsibly to the vendor. Help make our Internet a safer place :)*

Have you ever wondered how data breaches happen?

It seems like now in 2019, a new company is breached every five minutes. But how exactly do these breaches happen? How do hackers get their hands on sensitive data? I used to imagine hackers with hoodies and night goggles, breaking into company buildings in the night, James Bond style. But the reality is far less Holywood then that. Although break-ins like that do happen, one of the more common ways for a company to be breached is actually through software vulnerabilities. And today, we're gonna talk about a simple, yet very impactful vulnerability that attackers often use to gain access to sensitive customer data: IDOR.

## What is IDOR?

IDOR stands for "Insecure Direct Object Reference". And despite the long and sort of intimidating name, IDOR is actually a very simple vulnerability to understand. Essentially, just remember this: IDOR is missing access control.

Time for an example! Let's say *socialmedia.com* is a social media site that allows you to chat with others. And when you signed up, you noticed that your user ID on the website is *1234*. This website has a page that allows you to view all your messages with your friends located at the URL: *https://socialmedia.com/messages*. When you click on the "View Your Messages" button located on the homepage, you get redirected to:

```
https://socialmedia.com/messages?user_id=1234
```

Where you can see all your chat messages with your friends on the website. Now, what if you change the URL in the URL bar to the following?

```
https://socialmedia.com/messages?user_id=1233
```

You notice that you can now see all the private messages between another user (Whose user ID is 1233) and all his friends. Woah. What just happened? At this point, you have found an IDOR vulnerability.

![](https://vkili.github.io/blog/assets/images/idor-02.png)

## How IDORs work.

The reason you were able to see the messages of user 1233 is that there is no identity check in place before the server returns private info of users. The server was not verifying that you were, in fact, user 1233 or if you are an imposter. It simply returned the information, as you asked.

IDORs happen when access control is not properly implemented, and when the references to data objects (like a file or a database entry) are predictable. In this case, it was very easy to infer that you can retrieve the messages for user 1232 and user 1231

```
https://socialmedia.com/messages?user_id=1232
```

and

```
https://socialmedia.com/messages?user_id=1231
```

If the website were to use a unique, unpredictable key for each user, like

```
https://socialmedia.com/messages?user_key=6MT9EalV9F7r9pns0mK1eDAEW
```

Then, the website would not have been vulnerable. Because there is no way for an attacker to guess the value of *user_key*. But instead, the social media site implemented an insecure, direct, object reference.

These predictable "Direct Object References" exposes the data hidden behind them to the internet, allowing anyone to grab the information associated with the reference (in this case, the user's ID is used to reference the user's private messages).

## Implications of IDOR

As you can probably imagine, IDORs can become quite catastrophic for companies. In this case, think about all the bad PR *socialmedia.com *is gonna face when its users find out that anyone could have read their private messages!

This vulnerability could be exploited even more aggressively: attackers could simply write a script to query all user IDs and scrape all of the data automatically! If this vulnerability was to happen on an online shopping site, attackers might be able to harvest millions of bank accounts, credit card numbers, and addresses. If this vulnerability was to be found on a banking site, attackers could possibly leak everyone's credit information and tax forms! Talk about a scary vulnerability.

IDORs are not just limited to reading other users' information either. It can also be used to edit data on another user's behalf. Imagine this: you can change your own password on *socialmedia.com* by going to:

```
https://socialmedia.com/change_password?user_id=1234
```

And you can change the password of user number 1233 if you visit:

```
https://socialmedia.com/change_password?user_id=1233
```

This means that you could take over the account of any user on the platform by changing their passwords without their consent! IDORs on critical functionalities such as password reset, password change, and account recovery are critical vulnerabilities that compromise the entire web application.

On the other hand, even low impact IDORs can potentially be combined with other vulnerabilities to cause big issues. For example, let's say this link allows you to add a reminder for yourself on the site (eg. wish someone a happy birthday):

```
https://socialmedia.com/add_reminder?user_id=1234
```

And you found that this endpoint is vulnerable to IDOR. So what if you can add a reminder for someone else? No big deal right? They just might be a little freaked out. But what if the add reminder form is vulnerable to self-XSS? (Read more about self-XSS [here](https://en.wikipedia.org/wiki/Self-XSS).) An attacker can then turn the self-XSS into a [stored XSS](https://www.imperva.com/learn/application-security/cross-site-scripting-xss-attacks/), making it possible to steal everyone's session cookies! Then, it's game over for everyone's privacy!

## How to find IDORs

The reason why IDORs are so hard to prevent is that automatic vulnerability scanners are pretty bad at finding them. So the best way to discover IDORs is through a source code review to see if all direct object references are protected by access control.

Manual testing is also an effective way of testing for IDOR. When manual testing, you should create two different accounts and see if you can access the account info of the first account using the second account.

When testing, remember that IDORs can appear in URL parameters, form field parameters, file paths, headers, and cookies. Capture all the requests going between your web client and the web server. Inspect each of these requests carefully. Go through each request and always test the parameters that contain numbers, usernames or IDs.

Happy Hacking! Next time, we'll talk about how to find more IDORs and how to bypass common protection mechanisms.