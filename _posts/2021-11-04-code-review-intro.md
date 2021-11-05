---
title: "How To Review Code For Vulnerabilities"
categories:
  - Development
---

Performing a source code review is one of the best ways to find security issues in an application. But how do you do it?

In this tutorial, I will go through some tactics for performing a security code review on your application.

## Prerequisites

Before you start reviewing code, learn what the most common vulnerabilities are for the target application type. Getting familiar with the indicators and signatures of those vulnerabilities will help you identify similar patterns in source code. For example, the signature for an XXE vulnerability is passing user-supplied XML to a parser without disabling DTDs or external entities. (More about XXEs [here](https://blog.shiftleft.io/intro-to-xxe-vulnerabilities-appsec-simplified-80be40102815).)

Although vulnerabilities have common patterns in code, these patterns can look quite different in different contexts. Knowing the programming language, frameworks, and libraries you are working with will help you spot vulnerable patterns more accurately.

## Some Code Analysis Jargon

Before we go on, there are a few concepts that you should understand: "sources", "sinks", and "data flow". In code analysis speak, a "source" is the code that allows a vulnerability to happen. Whereas a "sink" is where the vulnerability actually happens.

Take command injection vulnerabilities, for example. A "source" in this case could be a function that takes in user input. Whereas the "sink" would be functions that execute system commands. If the untrusted user input can get from "source" to "sink" without proper sanitization or validation, there is a command injection vulnerability.

Many common vulnerabilities can be identified by tracking this "data flow" from appropriate sources to corresponding sinks.

## Quick Start Hunting

If you are short on time, focusing on a few issues can help you discover the most common and severe issues.

Start by searching for strings, keywords, and code patterns known to be indicators for vulnerabilities or misconfiguration. For example, hardcoded credentials such as API keys, encryption keys, and database passwords can be discovered by grepping for keywords such as "key", "secret", "password", or a regex search for hex or base64 strings. Don't forget to search in your git history for these strings as well.

Unchecked use of dangerous functions and outdated dependencies are also a huge source of bugs. Grep for dangerous functions and see if they are reachable by user-controlled data. For example, you can search for strings like and "system()" and "eval()" for potential command injections. Search through your dependencies to see if any of them they are outdated.

The "grepping" technique may help you identify vulnerabilities fast, but it cannot help you identify deeper and more nuanced vulnerabilities in your application. That's why you'll probably want to dig deeper into data flow analysis.

## Digging Deeper

You can complement the above strategy with a more extensive source code review if you have time.

Focus on areas of code that deal with user input. User input locations such as HTTP request parameters, HTTP headers, HTTP request paths, database entries, file reads, and file uploads provide the entry points for attackers to exploit the application's vulnerabilities. Tracing data flow from these functions to corresponding sinks can help you find common vulnerabilities such as stored-XSS, SQL injections, shell uploads, and XXEs.

Then, review code that performs critical functionalities in the application. This includes code that deals with authorization, authentication and other logic critical to business functions. Look at the protection mechanisms implemented and see if you can bypass them. At the same time, check how business and user data is being transported. Is sensitive information transported and stored safely?

Finally, look out for configuration issues specific to your application. Make sure that your application is using secure settings according to best practices.

## What about SAST?

As you can see, manual code review can be quite tedious and time-consuming. Using SAST (Static Analysis Security Testing) tools is a great way to speed up the process. Good SAST tools identify vulnerable patterns for you so that you can focus on analyzing the impact and exploitability of the vulnerability.

Automation can help you with other components of the security review as well. For example, SCA (Software Composition Analysis) tools automates the dependency tracking process. SCA tools keep track of an application's dependencies and alert you if new CVEs are found.

Code scanning tools are not a hundred percent accurate. So the best way to build secure software is to use tools to find possible vulnerabilities, then conduct a manual code review to validate them. This way, you can ensure that as few bugs make it to production as possible.

For more information about conducting code reviews, visit this guide: <https://owasp.org/www-pdf-archive/OWASP_Code_Review_Guide_v2.pdf>.