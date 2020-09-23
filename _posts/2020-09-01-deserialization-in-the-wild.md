---
title: "Deserialization Bugs In The Wild"
categories:
  - Insecure Deserialization
---

A totally unscientific analysis of deserialization vulnerabilities found in the wild.


## What is insecure deserialization?

Serialization is a process during which an object in a programming language (say, a Java object) is converted into a format that can be saved to the database or transferred over a network. Whereas deserialization refers to the opposite: it's when the serialized object is read from a file or the network and converted back into an object.

Insecure deserialization is a type of vulnerability that arises when an attacker is able to manipulate the serialized object and cause unintended consequences in the program's flow. This can cause DoS, authentication bypass or even RCE.

For example, if an application takes a serialized object from the user and uses the data contained in it to determine who is logged in, a malicious user might be able to tamper with that object and authenticate as someone who she is not. If the application uses an unsafe deserialization operation, the malicious user might even be able to embed code snippets in the object and get it executed during deserialization!

## Why they are interesting

1.  They are often critical vulnerabilities.

Very often, an insecure deserialization bug will result in code execution, granting attackers a wide range of capabilities to impact the application. As such, deserialization bugs are very valuable and impactful vulnerabilities.

2. They are hard to defend against.

Defending against deserialization vulnerability is difficult. How to protect an application against these vulnerabilities varies and depends greatly on the programming language, the libraries and the serialization format used. Because of this, there is no one-size-fits-all solution.

## Why I did this

Previously, I studied all the SSRF reports on HackerOne and found the experience to be extremely helpful. You can find the writeup [here](https://vickieli.dev/blog/ssrf/ssrf-in-the-wild/).

During the process of reading these reports, I gained a deeper understanding of the SSRF vulnerability class and was able to find a few bugs along the way.

Deserialization bugs have always fascinated me. I have read writeups from amazing researchers that utilized these issues to achieve RCEs on critical assets from companies such as Google and Facebook. These bugs are hard to find and exploit because they tend to look different depending on the programming language and library used. Thus, they require deep technical understanding and ingenuity to exploit. It would be really cool if I can find one in the wild myself.

So I figured, studying deserialization issues in a similar way will allow me to understand them more deeply and gain new security knowledge. And just maybe, even score me some RCEs?

In particular, I am interested in finding out:

1.  How are hackers finding deserialization issues?
2.  Where do these issues occur?
3.  What did the application do wrong to cause these issues?

## Getting started

In order to amass the raw material for my research, I started with three simple Google Dorks:

site:hackerone.com inurl:/reports/ "serialization"site:hackerone.com inurl:/reports/ "deserialization"site:hackerone.com inurl:/reports/ "object injection"

These three dorks yielded a total of 177 results at the time. These 177 links yielded 35 non-duplicate, relevant, real-world vulnerability reports. Others are either duplicate search results, CTF submissions, unexploitable issues, or vulnerabilities that are not actually caused by insecure deserialization.

![](https://vickieli.dev/blog/assets/images/serialize-01.png)

As you can tell, I was able to find a lot fewer reports than when I was searching for SSRFs. This was very much expected, and can be attributed to several factors:

1.  Deserialization bugs probably occur less commonly than SSRFs.
2.  Deserialization bugs generally require deeper technical knowledge to find than SSRFs.
3.  Companies might be reluctant to disclose these bugs because they are very high impact and often lead to RCE.

Enough said, let's dive into these publicly disclosed reports!

## The research process

When reading these reports, I mainly looked at a few things:

-   How did the researcher find the vulnerability? Was it found auditing source code directly or using a black-box approach?
-   What was the deserialization mechanism used? What were the language, library, and serialization format used?
-   How critical was the vulnerability? What can be done using this particular deserialization flaw? DoS, authentication bypass, or RCE? If RCE was possible, what was the privilege level of the current process? What can be done after achieving RCE?
-   What was the fix implemented by the vendor after the report?

I read each report and categorized them according to the above criteria. I also looked at the fixes implemented by the vendors, and if any bypasses were found after the initial fix.

## Results and analysis

Without further ado, here are the results I found from the publicly disclosed reports and writeups.

### How was the bug found?

For the majority of the reports, the hacker found the bug by analyzing source code directly. Some hackers looked at the code on the company's public Github repositories, while others obtained source code through other bugs like path traversal or SSRF.

![](https://vickieli.dev/blog/assets/images/serialize-02.png)

You can tell that source code review is still the most reliable way to detect deserialization vulnerabilities.

Although a lot of hackers were able to find deserialization vulnerabilities via black-box methodologies, these often took a lot more effort, time and luck.

### What was the vulnerable feature?

Features are potentially vulnerable to deserialization flaws if they deserialize objects supplied by the user. All of the disclosed reports reported bugs in typically vulnerable places: RPC, databases, web caches and authentication tokens/ HTML form parameters.

### What was the vulnerable function?

For most of the reports, I was able to determine the vulnerable function handling the deserialization. Here's a breakdown of the functions that led to the vulnerability:

![](https://vickieli.dev/blog/assets/images/serialize-03.png)

Most of the disclosed vulnerabilities were caused by PHP's unserialize(). Followed by the unsafe deserialization of Java objects, Ruby's Marshall.load() then Python's pickle.loads(). For the rest of the reports, there was not enough information to determine what the vulnerable function was.

### Interesting trends

Most of the PHP unserialize() bugs in this set was found around two to three years ago, whereas newer reports tend to be about deserialization issues in Java or Ruby.

Although, this might not mean anything since I only looked at 35 reports, reported by an even smaller number of hackers. This could be more indicative of reporter activity than actual vulnerability trends in the wild.

### What can be done using this particular deserialization flaw?

I also looked at the worst-case exploitation scenario as proposed by the hacker. Of the 35 reports, 33 of them included an attack scenario that leads to RCE, while others pointed to DoS and authorization bypass. (Note that reports that were about insecure coding patterns without an exploitable vulnerability were excluded from this study.)

![](https://vickieli.dev/blog/assets/images/serialize-04.png)

Most hackers included PoCs to proof RCE to back up their claims in the report. As you can see most of the disclosed deserialization vulnerabilities lead to RCE. And even if a deserialization bug does not lead to RCE, it can likely lead to other severe consequences.

I was not able to gather information about *what can be done* after achieving RCE since most hackers simply produced a harmless POC to avoid breaching the programs' rules of engagement.

### How critical was the vulnerability?

For this metric, I looked at two things: the final report state and the final severity rating.

![](https://vickieli.dev/blog/assets/images/serialize-05.png)

Of the 35 reports, only one was closed out as informative. All others resulted in a vendor fix and a "resolved" state.

The informative report was not closed as informative because the vulnerability was inconsequential, but because the affected component was maintained by a third-party and was out-of-scope per program terms.

I also looked at the final severity rating of the report. A large number of the reports were not given a severity rating. For the rest, most were rated as critical. Four reports were closed as "high" and three were closed as "medium".

![](https://vickieli.dev/blog/assets/images/serialize-06.png)

These reports were given a lower severity rating because of several reasons and mitigating factors:

-   RCE was not possible, and the hacker was only able to achieve DoS or authorization bypass.
-   The vulnerability was difficult to exploit and relied on an obscure point of entry.
-   The vulnerability required some level of privilege to exploit, and the vulnerable function was not available to unauthenticated users.

## Lessons learned

### Your next Github dork

A lot of the vulnerabilities were found simply by looking for the vulnerable functions in source code. So who knows, you might find your next RCE by Github searching for unserialize(), along with other known dangerous functions!

### Base64 decode --- everything!

Pay close attention to the large blobs of encoded data passed into an application and consider them for object injection opportunities. For example, Java serialized objects have the following signatures:

-   Starts with `AC ED 00 05` in Hex or `rO0` in Base64.
-   `Content-type` header of an HTTP response set to `application/x-java-serialized-object`.

### More public disclosure, please!

Though I was able to find many reports to read, a large portion of the disclosed reports I found was actually "limited disclosure". This means that only the report title and a summary were made available. I found a lot of the report titles and summaries intriguing, but they often did not provide enough details for others to learn from it.

So the next time you find a vulnerability, fully disclose it! Or, if the vendor does not agree to full disclosure, strive to write a summary that will allow others to gain knowledge from your discovery. Your efforts will be much appreciated!

## Conclusion

Coming out of my project, I feel like I now have a deeper understanding of deserialization vulnerabilities, as well as where and how they occur. Now, back to hacking!