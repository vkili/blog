---
title: "Intro to SSRF"
categories:
  - SSRF
---

And how your firewall failed you.

![](https://vickieli.dev/blog/assets/images/ssrf-01.png)

Successful cyberattacks often start at the "network perimeter".

As a company grows, it becomes increasingly difficult to secure the hundreds and thousands of machines on the network. Often, all an attacker needs to compromise a network is a single bug on a public-facing machine! Today, we will talk about a common vulnerability on the network perimeter: SSRF, how it allows attackers to pivot into a company's network, and how to find it.

## What is SSRF?

SSRF --- Server Side Request Forgery --- is a vulnerability that happens when an attacker is able to send requests on behalf of a server. It allows attackers to "forge" the request signatures of the vulnerable server, therefore assuming a privileged position on a network, bypassing firewall controls and gaining access to internal services.

For example, imagine that there is a public-facing web server on a company's network: *public.example.com*.

*public.example.com* hosts a proxy service located at *public.example.com/proxy* that would fetch the webpage specified in the "URL" parameter and display it back to the user. For example, when the user accesses the URL:

```
https://public.example.com/proxy?url=google.com
```

the web application would display the homepage of "google.com".

![](https://vickieli.dev/blog/assets/images/ssrf-02.png)

Now let's say *admin_panel.example.com* is an internal server hosting a hidden admin panel.

To ensure that only employees can access the admin panel, access control was set up so that the panel is not accessible via the Internet, but only from a valid internal IP (like an employee workstation). Now, what if a user accesses the following URL?

```
https://public.example.com/proxy?url=admin_panel.example.com
```

With no SSRF protection mechanism in place, the web application would display the admin panel back to the user, because the request is coming from *public.example.com*, a trusted machine on the network!

![](https://vickieli.dev/blog/assets/images/ssrf-03.png)


Unauthorized requests that would normally be blocked by firewall controls (like fetching the admin panel from a non-company machine) is now allowed. This is because the protection that exists on the network perimeter --- between public-facing web servers and Internet machines --- does not exist between machines on the trusted network --- between *public.example.com* and *admin_panel.example.com*.

![](https://vickieli.dev/blog/assets/images/ssrf-04.png)

Using the ability to "forge" requests from trusted servers, an attacker can now conduct all kinds of nasty shenanigans on the network. Depending on the permissions given to the vulnerable server, an attacker might be able to read sensitive files, make internal API calls and access internal services like hidden admin panels.

There are two types of SSRF vulnerabilities: regular SSRF and blind SSRF. The mechanisms behind both vulnerabilities are the same: they both exploit the trust between machines on the same network. The only difference is that in blind SSRFs, the attacker does not receive feedback from the server via an HTTP response or an error message (like how *admin_panel.example.com* was displayed in the example above). Although this makes data exfiltration and network exploration harder, blind SSRFs are still extremely valuable for an attacker. We'll get more into this later!

## How to test for SSRFs

The best way to discover SSRF vulnerabilities is a manual code review to see if all URL inputs are being validated. However, when source code is not available and when a full code review is not possible, efforts should be focused on testing the features that are most prone to SSRF.

SSRFs occur when a server requires external resources. For example, sometimes a web application would need to create a thumbnail from a URL of an image, or create a screenshot of a video from another site (like youtube.com). If a server doesn't restrict access to internal resources, SSRF vulnerabilities occur.

The following page on *public.example.com *allows users to upload a profile photo from the Internet:

```
https://public.example.com/upload_profile_from_url.php?url=www.google.com/cute_pugs.jpeg
```

In order to fetch *cute_pugs.jpeg* from *google.com*, the web application would have to visit and retrieve contents from *google.com*. If the server doesn't make a distinction between internal and external resources, an attacker could just as easily request:

```
https://public.example.com/upload_profile_from_url.php?url=localhost/secret_password_file.txt
```

And make the webserver display the file that contains the password to the webserver.

Features that are often vulnerable to SSRF include webhooks, file upload via URL, document and image processors, link expansion, and proxy services (these features all require visiting and fetching external resources). However, it is worth testing any endpoint that processes a user-provided URL.

Tests for SSRF usually start by providing the URL input with an internal address. Depending on the network configuration, you might need to try several different addresses. The following are a few to try first:

```
-   127.0.0.0/8
-   192.168.0.0/16
-   10.0.0.0/8
```

Here's a [link](https://en.wikipedia.org/wiki/Reserved_IP_addresses) to other reserved IP addresses.

In the case of regular SSRF, see if the server is returning a response that reveals any information about the internal service. Does the response contain service banners or HTML content of internal pages?

For example, when you request:

```
https://public.example.com/upload_profile_from_url.php?url=127.0.0.1:22
```

The server responds with:

```
Error: cannot upload image: SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.4
```

In the case of blind SSRF, try to determine if there is a difference in server behavior between commonly open and closed ports (ports 80 and 443 are commonly open ports, while port 11 is not). Look out especially for differences in response time and HTTP response codes.

For example, the following request results in an HTTP status code of 200 (Status code for "OK").

```
https://public.example.com/webhook?url=127.0.0.1:80
```

While the following request results in an HTTP status code of 500 (Status code for "Internal Server Error").

```
https://public.example.com/webhook?url=127.0.0.1:11
```

We can confirm that SSRF exists and deduce that port 80 is open, and port 11 is closed on the server.

Here's a [link](http://packetlife.net/media/library/23/common-ports.pdf) to some common ports and their associated services.

Thanks for reading. Please help make this a better resource for new hackers: feel free to point out any mistakes or let me know if there is anything I should add!