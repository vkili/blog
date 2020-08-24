---
title: "Bypassing SSRF Protection"
categories:
  - SSRF
---

There's always more to do...

![](https://vkili.github.io/blog/assets/images/ssrf-07.png)

Ok. So you've found a feature on a web application that fetches external resources. You're able to pull content from all sorts of external sites and there doesn't seem to be any restrictions on the file type that you can request... The application displays everything right back at you. Everything on this endpoint is screaming at you, that it is ripe for SSRF! So you start typing in the magic digits: 127.0.0.1. But a mere second later, the server comes back with an unexpected response:

```
Error. Requests to this address are not allowed. Please try again.
```

Well, frak. What do you do now?

## SSRF Protection Mechanisms

Companies have really caught onto the risk of SSRF attacks. As a result, most have implemented some form of SSRF protection on their web applications. There are two main types of SSRF protection mechanisms out there: blacklists and whitelists.

Blacklists refer to the practice of not allowing certain addresses and blocking the request if a blacklisted address was received as input. Most SSRF protection takes the form of blacklisting internal network address blocks.

On the other hand, whitelists mean that a server would only allow through requests that contain URLs on a prespecified list and fail all other requests.

## Bypassing Whitelists

Whitelists are generally harder to bypass because they are by default, stricter than blacklists. But it is possible if there is an open redirect vulnerability within the whitelisted domains.\
If you could find an open redirect, you can request a whitelisted URL that redirects to an internal URL.

If the whitelist is not correctly implemented (eg. via poorly designed regex), it could also be bypassed by using making a subdomain or directory as the whitelisted domain name (eg. *victim.com.attacker.com* or *attacker.com/victim.com*).

![](https://vkili.github.io/blog/assets/images/ssrf-08.png)

## Bypassing Blacklists

However, due to application requirements (fetching external resources), most SSRF protection mechanisms come in the form of a blacklist. If you are faced with a blacklist, there are numerous ways of tricking the server:

### Fooling it with redirects

Make the server request a URL that you control that redirects to the blacklisted address. For example, you can host a file with the following content on your web server:

```php
<?php header("location: http://127.0.0.1"); ?>
```

Let's say this file is hosted at *http:*//*attacker.com/redirect.php*. This way, when you make the target server request *http:*//*attacker.com/redirect.php*, the target server is actually redirected to http://127.0.0.1, a restricted internal address.

### Tricking it with DNS

Modify the A/AAAA record of a domain you control and make it point to internal addresses of the victim's network. For example, let's say *http://attacker.com* is a subdomain that you own. You can create custom hostname to IP address mapping and make *http://subdomain.attacker.com* resolve to 127.0.0.1. Now when the target server requests *http://attacker.com*, it would think that your domain is located at 127.0.0.1 and request data from that address!

### Using IPv6 addresses

Try using IPv6 addresses instead of IPv4. The protection mechanisms implemented for IPv4 might not have been implemented for IPv6.

### Switching out the encoding

There are many different ways of encoding a URL or an address that doesn't change how a server interprets its location, but might let it slip under the radar of a blacklist. These include hex encoding, octal encoding, dword encoding, URL encoding, and mixed encoding.

-   Hex Encoding

Hexadecimal encoding is a way of representing characters in the base 16 format (characters range from 0 to F) as opposed to base 10 (decimal, characters range from 0 to 9). Turns out that servers out there can understand IP addresses when they are hex encoded. To translate a decimal formatted IP address to hex, you need to calculate each section of the IP address into its hex equivalent. (Or maybe just use a calculator...) Here's an example:

```
127.0.0.1 translates to 0x7f.0x0.0x0.0x1
```

The "0x" at the beginning of each section designates it as a hex number.

-   Octal Encoding

Octal encoding is a way of representing characters in the base 8 format. Similar to how you would translate an address to the hex format, you translate an IP address to octal form by recalculating each section of the IP address. For example,

```
127.0.0.1 translates to 0177.0.0.01
```

In this case, the leading zeros are necessary to convey that that section is an octal number.

-   Dword Encoding

"Dword" stands for "double word", which is a 32-bit integer. An IP address is basically a 32-bit number, split into four octets (groups of eight bits), and written in the decimal format. For example, 127.0.0.1 is actually the decimal representation of 01111111.00000000.00000000.00000001. And when we translate the entire number (01111111000000000000000000000001) into one single decimal number, we get the IP address in dword format!

So what is 127.0.0.1 in dword? It's the answer for 127*256³+0*256²+0*256¹+1*256⁰, which is 2130706433. (Whip out that scientific calculator! Or just use a decimal to binary calculator.) Meaning that if you type in http://2130706433 instead of http://127.0.0.1, it would still be understood! Pretty cool right?

-   URL Encoding

Each individual character in a URL can be represented by their designated hex number, if they are preceded by a % symbol. For example, the word "localhost" can be represented with its URL encoded equivalent, "%6c%6f%63%61%6c%68%6f%73%74". So when a server is blocking requests to internal hostnames such as "localhost", try it's URL encoded equivalent!

-   Mixed Encoding

It's mix-and-match time! You could also use a combination of encoding techniques to try to fool the server: maybe this would work?

```
127.0.0.1 translates to 0177.0.0.0x1
```

## Conclusion

This is just a small portion of bypasses that an attacker could have in their arsenal, and I'm pretty sure that there are many more creative ways out there to defeat protection and achieve SSRF.

When you can't find a bypass that works, it helps to switch your perspective and think: how would I implement the SSRF protection mechanisms for this feature? Design and write down what you think the protection logic would look like. Then, try to bypass the protection mechanism that you have designed. Is it possible? Did you miss anything when implementing the protection? Is it possible that the developer of the application has also missed something?

## Happy Hacking!

Next time, we'll talk about some interesting cases of SSRFs found in the wild.