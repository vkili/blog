---
title: "Intro to CSRF: Cross-Site Request Forgery"
categories:
  - CSRF
---

To change your password, password not required!

Hello! And welcome to this blog post!

I'm excited to say that today, we are going to dive into the first vulnerability type that I learned about when I started in web hacking: CSRF, or Cross-Site Request Forgery.

## What is CSRF?

CSRF, or Cross-Site Request Forgery, is a technique that allows hackers to carry out unwanted actions on a victim's behalf. Think: a hacker changing your password or transferring money from your bank account without your permission.

Sounds crazy and pretty cool already! But how is this attack possible? Before we dive into the mechanisms of a CSRF attack, we need to understand how websites authenticate their users.

## How does Twitter know it's me?

Have you ever wondered: How do websites recognize me? Why is it that I don't need to log into Twitter every time I check my feed? When I Tweet something, how does Twitter know it's me without having to ask me for my password again?

Once you log in to a website, the browser that you are using stores your session cookies for that website, and sends it along automatically every time you communicate with the site.

For example, after you log into Twitter, Twitter issues a session cookie for your account. That session cookie authenticates you to the website. The browser that you are using receives the session cookie, stores it, and sends it along with every request to Twitter. This allows you to access confidential information only available to you, and perform actions that only you should be able to do, like reading your DMs and changing your account information.

Thus, when you send a Tweet out into the world: your browser sends a request to Twitter with your session cookie, proving your identity, thus verifying that you are authorized to send the Tweet as your username.

Let's say the "Send a Tweet" HTML form looks like this: (This is not the actual request used by Twitter and the Twitter "Send a Tweet" functionality is not actually vulnerable to CSRF attacks.)

```html
<form method="POST" action="https://twitter.com/send_a_tweet">
 <input type="text" name="tweet_content" value="Hello world!">
 <input type='submit' value="Submit">
</form>
```

Once you click on "Submit" on the page, your browser will send a request to https://twitter.com/send_a_tweet with your session cookie, and a tweet would be sent on your behalf. Now, can you spot the issue with this functionality?

## How it all goes wrong...

A simple HTML form like the above is vulnerable because any site can submit that same request, not just twitter.com.

Imagine an attacker hosting her own website, and on her site, she hosts an auto-submitted and invisible HTML form like this:

```html
<form method="POST" action="https://twitter.com/send_a_tweet">
 <input type="text" name="tweet_content" value="Follow @vickieli7 on Twitter!">
 <input type='submit' value="Submit">
</form>
```

Since browsers include session cookies automatically with each request, and this request is going to twitter.com, the browser will include your Twitter credentials when this form is automatically submitted.

Twitter will then see the request as valid since it included your real Twitter credentials. The Tweet will go through and you would have been forced to tweet: Follow @vickieli7 on Twitter! whenever you visit the malicious page.

Realistically, a malicious HTML page would look like this: The form is hidden from user view and Javascript is used to submit the form without the need for user interaction.

```html
<iframe style="display:none" name="csrf-frame"></iframe>
<form method="POST" action="https://twitter.com/send_a_tweet" target="csrf-frame" id="csrf-form">
 <input type="text" name="tweet_content" value="Follow @vickieli7 on Twitter!">
 <input type='submit' value="Submit">
</form>
<script>document.getElementById("csrf-form").submit()</script>
```

## How to protect yourself against CSRF attacks

Unfortunately, CSRF vulnerabilities are still extremely common in the wild. Websites of major corporations like Starbucks and Uber have all been found vulnerable to the attack. While developers can do their best to protect their websites, we as users can also take action to minimize the risk of a CSRF attack.

Here are a few steps you can take to minimize your risk:

-   Log out of sites after using: CSRF attacks rely on the victim having a valid session for a particular site. When you log out, you invalidate the session on that site, leaving no sessions for the hacker to exploit. (The cookies stored in your browser will no longer work.)
-   Use a different browser for sensitive sites: CSRF attacks also rely on the victim's browser storing the credentials of the vulnerable website. By using separate browsers for sensitive activities (such as banking sites) and casual browsing, you can make sure that if you do stumble upon a CSRF exploit page during casual browsing, the browser that you are using will not have the credentials to sensitive sites.
-   Use a Javascript blocker: JavaScript is often used to automatically submit the forms used to exploit a CSRF vulnerability. By disabling Javascript on your browser or email client, you essentially render auto-submit forms useless. This way, CSRF attacks would become harder since attackers would have to trick you into manually submitting the form instead.