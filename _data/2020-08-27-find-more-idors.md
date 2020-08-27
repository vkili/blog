---
title: "How to Find More IDORs"
categories:
  - Info Leak
---
And maximize their impact while hunting for bugs.

I love IDORs. They are easy to find, simple to exploit and often carry significant business impact. They were the root causes of some of the most critical vulnerabilities that I've found.

If you aren't familiar with IDORs or need a refresher, you can find an article explaining the basics of IDORs [here](https://vkili.github.io/blog/info%20leak/intro-to-idor/).

However, IDORs aren't always as simple as decrementing and switching out a numeric ID. As applications become more functionally complex, the way they reference resources often become more complex as well. This means that simple, numeric IDORs are becoming rarer in most web applications.

IDORs are manifesting in applications in different ways. And here are a few places to pay attention to, beyond your plain old numeric IDs.

## Unsuspected places to look for IDORs

### Don't ignore encoded and hashed IDs

When faced with an encoded ID, it might be possible to decode the encoded ID using common encoding schemes.

And if the application is using a hashed/ randomized ID, see if the ID is predictable. Sometimes applications use algorithms that produce insufficient entropy, and as such, the IDs can actually be predicted after careful analysis. In this case, try creating a few accounts to analyze how these IDs are created. You might be able to find a pattern that will allow you to predict IDs belonging to other users.

Additionally, it might be possible to leak random or hashed IDs via another API endpoint, on other public pages in the application (profile page of other users, etc), or in a URL via referer.

For example, once I found an API endpoint that allows users to retrieve detailed direct messages through a hashed conversation ID. The request kinda looks like this:

```http
GET /api_v1/messages?conversation_id=SOME_RANDOM_ID
```

This seems okay at first glance since the *conversation_id *is a long, random, alphanumeric sequence. But I later found that you can actually find a list of conversations for each user just by using their user ID!

```http
GET /api_v1/messages?user_id=ANOTHER_USERS_ID
```

This would return a list of *conversation_ids* belonging to that user. And the *user_id* is publicly available on each user's profile page. Therefore, you can read any user's messages by first obtaining their user_id on their profile page, then retrieving a list of conversation_ids belonging to that user, and finally loading the messages via the API endpoint /api_v1/messages!

### If you can't guess it, try creating it

If the object reference IDs seem unpredictable, see if there is something you can do to manipulate the creation or linking process of these object IDs.

### Offer the application an ID, even if it doesn't ask for it

If no IDs are used in the application generated request, try adding it to the request. Try appending *id, user_id, message_id* or other object reference params and see if it makes a difference to the application's behavior.

For example, if this request displays all your direct messages:

```http
GET /api_v1/messages
```

What about this one? Would it display another user's messages instead?

```http
GET /api_v1/messages?user_id=ANOTHER_USERS_ID
```

### HPP (HTTP parameter pollution)

HPP vulnerabilities (supplying multiple values for the same parameter) can also lead to IDOR. Applications might not anticipate the user submitting multiple values for the same parameter and by doing so, you might be able to bypass the access control set forth on the endpoint.

Although this seems to be rare and I've never seen it happen before, theoretically, it would look like this. If this request fails:

```http
GET /api_v1/messages?user_id=ANOTHER_USERS_ID
```

Try this:

```http
GET /api_v1/messages?user_id=YOUR_USER_ID&user_id=ANOTHER_USERS_ID
```

Or this:

```http
GET /api_v1/messages?user_id=ANOTHER_USERS_ID&user_id=YOUR_USER_ID
```

Or provide the parameters as a list:

```http
GET /api_v1/messages?user_ids[]=YOUR_USER_ID&user_ids[]=ANOTHER_USERS_ID
```

### Blind IDORs

Sometimes endpoints susceptible to IDOR don't respond with the leaked information directly. They might lead the application to leak information elsewhere instead: in export files, emails and maybe even text alerts.

### Change the request method

If one request method doesn't work, there are plenty of others that you can try instead: GET, POST, PUT, DELETE, PATCH...

A common trick that works is substituting POST for PUT or vice versa: the same access controls might not have been implemented!

### Change the requested file type

Sometimes, switching around the file type of the requested file may lead to the server processing authorization differently. For example, try adding .json to the end of the request URL and see what happens.

## How to increase the impact of IDORs

### Critical IDORs first

Always look for IDORs in critical functionalities first. Both write and read based IDORs can be of high impact.

In terms of state-changing (write) IDORs, password reset, password change, account recovery IDORs often have the highest business impact. (Say, as compared to a "change email subscription settings" IDOR.)

As for non-state-changing (read) IDORs, look for functionalities that handle the sensitive information in the application. For example, look for functionalities that handle direct messages, sensitive user information, and private content. Consider which functionalities on the application makes use of this information and look for IDORs accordingly.

### Stored-XSS

When you combine write-IDOR with self-XSS, you can often create a stored-XSS targeted towards a specific user.

When would this be useful? Let's say you find an IDOR that allows attackers to change the content of another user's internet shopping list. This IDOR in itself would not be too high impact, and would likely just cause a bit of annoyance if exploited in the wild. But if you can chain this IDOR with a self-XSS on the same input field, you can essentially use this IDOR to deliver the XSS exploit code to the victim user's browser. This way, you can create targeted stored-XSS that requires no user interaction!

Thanks for reading. Please help make this a better resource for new hackers: feel free to point out any mistakes or let me know if there is anything I should add!