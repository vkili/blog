---
title: "Designing Secure Software: A Guide for Developers"
categories:
  - Book reviews
---

AppSec engineer's book club #001 --- discussing Loren Kohnfelder's book

Many of my followers have been asking me for book recommendations. After all, who doesn't love a new tech book? Books are my favorite way to absorb new information, especially when learning something new.

I've wanted to start a security engineer's book club to share book recommendations, notes, and summaries for a while now. Over the past year, I've read quite a few good technical and non-technical books, including the following:

-   "Designing Secure Software: A Guide for Developers" by Loren Kohnfelder
-   "Container Security: Fundamental Technology Concepts that Protect Containerized Applications" by Liz Rice
-   "Hacking APIs: Breaking Web Application Programming Interfaces" by Corey J. Ball
-   "Bug Bounty Bootcamp: The Guide to Finding and Reporting Web Vulnerabilities" by Vickie Li :)
-   "The Staff Engineer's Path: A Guide for Individual Contributors Navigating Growth and Change" by Tanya Reilly, and more.

I keep a few next to my desk because I constantly find myself referencing them. In these articles, I hope to share with you some of the best engineering books I've read in the past year and hear from you about your favorite tech books.

Today, let's discuss Loren Kohnfelder's "Designing Secure Software: A Guide for Developers." And as with all No Starch Press titles, the book's cover art is ridiculously good.

![](https://miro.medium.com/max/344/0*XEjDAof1U2oOL1wC.jpg)

## Summary

"Designing Secure Software" has three parts: concepts, design, and implementation. "Concepts" goes through fundamental security concepts, like CIA (confidentiality, integrity, and availability), how to threat model an application, and some structural ways of securing an application (minimize data exposure, etc.). It also talks about secure design patterns and using cryptography safely.

"Design" goes into how to think about design from a security perspective and do security design reviews. This is an essential skill for many product security engineers, and I find myself referencing it repeatedly.

"Implementation" talks about secure programming and avoiding common implementation pitfalls like input issues that lead to vulnerabilities and SOP (Same-Origin Policy) issues. This part of the book also talks about writing security tests.

This book is one of the tech books I enjoyed reading last year. Technical books that read smoothly are rare, and this is one of them. The book provides a good overview of the application security field and gives you the resources to explore more.

One thing I found disappointing was that the chapter on cryptography felt cursory. Since the author is the inventor of digital certificates, I came into the book with the presumption that he would dive deeper into the topic.

## How To Read This Book

This is not a technical reference book but an "intro to product security" college course. The book is meant for software engineers who want to learn more about security or security professionals diving into application security.

The author introduces most major skills in application security at a high level, so there are more effective ways of learning from it than simply reading through the book. I would suggest taking notes of key points while reading and taking the time to practice applying the concepts instead.

For example, when reading about security requirements, try listing security requirements in a project you are working on in addition to understanding the examples in the book.

### Assignment list

The book has a list of exercises that supplement the concepts it introduces. Here are my picks of exercises from the book. Feel free to try them out. They will help you improve your skills in appsec even if you have not read the book.

-   Threat model a piece of software or a component of the larger system. For bonus points, threat model the component using different threat modeling frameworks and see if your results differ.
-   Retroactively create a design doc for a piece of software you created or use and review it from a security perspective. Identify components in the software where security matters the most and assess its security strengths and weaknesses. Suggest mitigation or alternative designs based on your findings.
-   To get a feel of realistic vulnerabilities, participate in open-source projects' bug bounty or vulnerability disclosure programs. Identify the root cause of the bug and suggest code changes to fix the bug.
-   Identify locations of untrusted input on the same open-source project and test whether it's protected against XSS, SQL injections, and path traversal.
-   Check out an old version of open-source software with a known vulnerability. Write a test that confirms the vulnerability.

You can always design your own assignments from the concept you've learned. Or perhaps, design a "college course" curriculum with an assignment list based on the book and teach product security to yourself.

> "If debugging is the process of removing software bugs, then programming must be the process of putting them in."
>
> -- Edsger W. Dijkstra

That's it for today's article! Writing truly is a muscle that atrophies. I regret not blogging for this long, but it feels good to be back. Thanks for reading, as always.