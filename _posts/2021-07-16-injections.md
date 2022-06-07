---
title: "They are all Injection Vulnerabilities!"
categories:
  - Hacking
---

What do SQL injections, command injections, and cross-site scripting have in common? The answer is that they are all injection vulnerabilities!

Lately, I have been thinking a lot about how to teach security. And something I think is really important when learning about security is understanding the fundamentals of why something is happening. So instead of learning about a singular technique or vulnerability class, you want to understand the underlying mechanisms of what caused these issues and why a certain attack is working the way it is.


## The fundamentals of injections

Injection issues are super common. They are the underlying issue for a huge percentage of vulnerabilities found in real-world applications. So today, let’s dive into the fundamental mechanisms of injection flaws.

In a single sentence, injections happen when applications cannot properly distinguish between untrusted user data and code.

## Untrusted user data

So the first piece of the equation is untrusted user data. Untrusted user data can come in the form of HTTP request parameters, HTTP headers, and cookies. They can also come from databases or stored files that can be modified by the user. If the application does not properly process the untrusted user data before inserting it into a command or query, the program’s interpreter will confuse the user input as a part of a command or a query. In this case, attackers can send data to an application in a way that will change the meaning of its commands.

In a SQL injection attack, for example, the attacker injects data to manipulate SQL commands. And in an XSS attack, the attacker injects data that manipulates the logic of scripts in HTML documents. So Any program that combines user data with programming commands or code is potentially vulnerable.

Also, it’s important to remember that this manipulation can happen any time this piece of data is being processed or used. Even if the malicious user data is not used by the application right away, the untrusted data can eventually travel somewhere in the program where it can do something bad, such as a dangerous function or an unprotected query. And this is where they cause damage to the application, its data, or its users.

##Preventing injection vulnerabilities

That’s why injections are so difficult to prevent. Untrusted data can attack any application component that it touches down the stream. And for every piece of untrusted data the application receives, it needs to detect and neutralize attacks targeting every part of the application. And application might think a piece of data is safe because it does not contain any special characters used for triggering XSS when the attacker intends to trigger an SQL injection instead. It’s not always straightforward to determine what data is safe and data is not, because safe and unsafe data looks very different in different parts of the application.

#### Input validation

So how do you protect against these threats? The first thing you can do is to validate the untrusted data. This means that you either implement a blocklist to reject any input that contains dangerous characters that might affect application components. Or you implement an allowlist that only allows input strings with known good characters. For example, let’s say that you are implementing a sign-up functionality. Since you know that the data is going to be inserted into a SQL query, you reject any username input that are special characters in SQL, like the single quote. Or, you can implement a rule that only allows alphanumeric characters.

But sometimes blocklists are hard to do because you don’t always know which characters are going to be significant to application components down the line. If you just miss one special character, that can allow attackers to bypass protection.

And allowlists may be too restrictive and in some cases, and sometimes you might need to accept special characters like single quotes in user input fields. For example, if a user named Conan O’Brien is signing up, he should be allowed to use a single quote in his name.

#### Parameterization

Another possible defense against injection is parameterization. Parameterization refers to compiling the code part of a command before any user-supplied parameters are inserted.

This means that instead of concatenating user input into program commands and sending it to the server to be compiled, you define all the logic first, compile it, then insert user input into the command right before execution. After the user input is inserted into the final command, the command will not be parsed and compiled again. And anything that was not in the original statement will be treated as string data, and not executable code. So the program logic part of your command will remain intact.

This allows the database to distinguish between the code part and the data part of the command, regardless of what the user input looks like. This method is very effective in preventing some injection vulnerabilities, but cannot be used in every context in code.

#### Escaping

And finally, you can escape special characters instead. Escaping means that you encode special characters in user input so that they are treated as data and not as special characters. By using special markers and syntax to mark special characters in user input, escaping lets the interpreter know that the data is not intended to be executed.

But this method comes with its problems as well. For one, you must use the exact encoding syntax for every downstream parser or risk the encoded values being misinterpreted by a parser. You might also forget to escape some characters, which attackers can use to neutralize your encoding attempts. So a key to preventing injection vulnerabilities is to understand how parsers of different languages work, and which parsers run first in your processes.

And that’s it for today’s security lesson. This blog post might be a bit general but understanding the fundamentals of injection vulnerabilities will give you a good foundation to start studying more complex and weird cases of injection vulnerabilities. So I hope this helps you form an intuition of how injection attacks and protections work. Thanks for reading!

What other security concepts do you want to learn about? I’d love to know. Feel free to connect on Twitter [@vickieli7](https://twitter.com/vickieli7).