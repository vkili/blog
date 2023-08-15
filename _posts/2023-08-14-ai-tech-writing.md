---
title: "Tech Writing vs GPT"
categories:
  - AI
---

Is AI replacing technical writers and developer advocates?

I recently stumbled on a Twitter discussion debating whether the role of developer relations (DevRel) / developer advocates would become obsolete due to generative AI, such as GPT.

DevRels are the people who create and maintain a community around a product. Their responsibilities include writing documentation and blog posts, producing educational content like videos, newsletters, and podcasts, and organizing or participating in tech events.

Although it's unlikely that the face-to-face aspect of DevRel can be replaced anytime soon, a big part of many DevRel positions is generating digital content: specifically technical writing, such as external-facing documentation and blogs. For example, while working in both DevRel and technical writing, I frequently had to explain security concepts in-depth to software engineers. So I decided to prompt ChatGPT to create a blog post about XSS and see how it does.

```
Create a blog post that explains XSS vulnerabilities.\
The audience of the blog post should be software engineers.\
The post should include different types of XSS,\
  and how to write XSS safe code,\
  with both examples of vulnerable code and how to fix them.
```

ChatGPT generated this article. (Feel free to skim and skip forward to my commentary below.)

> Start ChatGPT Article

```Protect Your Web Application: Understanding and Preventing Cross-Site Scripting (XSS) Vulnerabilities```

*Introduction*

*As software engineers, we all want to create web applications that are both functional and secure. However, one of the most dangerous security threats that can impact a web application is Cross-Site Scripting (XSS) vulnerabilities. In this blog post, we will explore the different types of XSS vulnerabilities and discuss best practices for writing XSS-safe code.*

*Types of XSS Vulnerabilities*

*XSS vulnerabilities can be categorized into three main types: Reflected XSS, Stored XSS, and DOM-Based XSS. Reflected XSS happens when user input is reflected back to the user in an unescaped form, allowing malicious code to be injected. Stored XSS, on the other hand, occurs when malicious code is injected into a database and is served to all users who access the affected page. Finally, DOM-Based XSS targets the client-side scripts that manipulate the Document Object Model (DOM).*

*Risks of XSS Vulnerabilities*

*XSS vulnerabilities can lead to several risks, including data theft, malware distribution, and website defacement. When a web application is vulnerable to XSS attacks, an attacker can steal sensitive data, execute malicious scripts, or even take over the entire website. Some examples of real-world incidents involving XSS vulnerabilities include the MySpace worm and the Samy worm.*

*Best Practices for Writing XSS Safe Code*

*To prevent XSS vulnerabilities, you should implement input validation and sanitization, output encoding and escaping, and the proper use of cookies. Let's take a closer look at each of these practices:*

1.  *Input Validation and Sanitization: Validate all user input to ensure it meets expected criteria and sanitize it to remove any malicious code.*
2.  *Output Encoding and Escaping: Encode all output to prevent malicious code from being injected into a web page. Escaping special characters like <, >, and & is also crucial.*
3.  *Proper Use of Cookies: Use HTTPOnly and Secure cookies to prevent cookie theft and ensure that cookies are only transmitted over encrypted connections.*

*Examples of Vulnerable Code and How to Fix Them*

*Here are some examples of vulnerable code and how to fix them:*

*Vulnerable Code:*

```js
document.write("<h1>Welcome, " + username + "!</h1>");```

*This code is vulnerable to Reflected XSS attacks because the variable *`*username*`* is not sanitized.*

*Fixed Code:*

```js
document.write("<h1>Welcome, " + escapeHTML(username) + "!</h1>");```

*The *`*escapeHTML()*`* function will encode special characters in the *`*username*`* variable to prevent XSS attacks.*

*Vulnerable Code:*

```html
<input type="text" value="<%= user.name %>">```

*This code is vulnerable to Stored XSS attacks because the *`*user.name*`* variable is not sanitized.*

*Fixed Code:*

```html
<input type="text" value="<%= escapeHTML(user.name) %>">```

*Using the *`*escapeHTML()*`* function will encode special characters in the *`*user.name*`* variable to prevent XSS attacks.*

*Tools for Identifying and Preventing XSS Vulnerabilities*

*Several tools are available to help identify and prevent XSS vulnerabilities, such as static and dynamic analysis tools, penetration testing, and third-party security services. It's important to incorporate these tools into your web application development process to ensure your code is secure.*

*Conclusion*

*XSS vulnerabilities are a serious threat to the security of web applications, but by following best practices for writing XSS-safe code and using available tools, we can prevent them from being exploited. Remember to always validate and sanitize user input, encode output, and use secure cookies to protect your web application from XSS attacks.*

> End ChatGPT Article

Initially, the article looks solid. It covers a lot of topics related to XSS that readers may find useful. However, I can't say that ChatGPT is a great technical writer. It missed out on a lot of things that make tutorials or documentation helpful.

## Anticipating a reader's needs

The majority of mental labor that goes into writing is anticipating the audience's needs, and figuring out what they came here for. For example, you'd need to understand your audience, and what prior knowledge they are likely to have, and decide which concepts to introduce, in what order, and to what depth. You'll also need to predict what trouble spots your audience may run into in the article, and build up to complex topics incrementally.

For instance, in the "Types of XSS Vulnerabilities" section, the generated article states that "*Reflected XSS happens when user input is reflected back to the user in an unescaped form, allowing malicious code to be injected.*"

What does "reflected back" mean? What type of malicious code should one inject? Would someone unfamiliar with XSS understand these terms intuitively? A good technical article should bridge gaps between topics and introduce new technical content based on pre-existing knowledge, which is what I found the most lacking in the generated article.

## Depth of understanding and research

The AI introduced a very high-level overview of the topic but lacked direction on which details to include. It would take someone with more in-depth knowledge of XSS to provide these follow-up prompts. The depth of knowledge provided by GPT-generated content is directly related to the prompter's understanding of the topic.

For example, for an article targeted toward software engineers, the article's discussion about prevention and sanitization is too shallow. To make the article more useful, I'll need to ask ChatGPT to discuss what these sanitization functions do, how they prevent certain types of XSS, and if there are attacker techniques that can bypass the protection.

## Forming opinions

When asked about the best practices to prevent XSS in certain contexts, ChatGPT struggled to provide a well-argued response, and simply stated that "*All of the methods mentioned in the blog post --- input validation and sanitization, output encoding and escaping, and proper use of cookies --- are important and should be implemented together to prevent XSS vulnerabilities effectively.*"

Although GPT should have a much deeper reserve of XSS knowledge than me, this advice sounds like it came from someone with a shallow understanding of the topic. Forming technical opinions requires an understanding of the broader context around the technical problem and making generalizations based on multiple scenarios. GPT does not have this capability.

## Bottom line

AI cannot fully replace human writers or authors --- not yet, at least. There are many things that ChatGPT is good at, but writing is not one of them.

ChatGPT could make writers more efficient by helping them create outlines and introductions, and edit their work. However, ChatGPT is not a great technical writer by itself, and creating a good article with ChatGPT would likely require a good human writer as the prompter.

Nonetheless, it's likely that its writing abilities will improve in the near future. For now, I'll just use GPT as a writing and research assistant.

*This article was written by a human and copyedited by ChatGPT. Want to connect? What other security concepts do you want to learn about? I'd love to know. Feel free to connect on Twitter [@vickieli7](https://twitter.com/vickieli7).-