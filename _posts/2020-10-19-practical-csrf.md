---
title: "Attacking Sites Using CSRF"
categories:
  - CSRF
---

From CSRF to user information leak, XSS and full account takeover.

The criticality of a CSRF vulnerability depends heavily on where the vulnerability is located. Sometimes, faulty CSRF protection mechanisms lead to inconsequential issues like unauthorized setting changes or emptying a user's cart. Other times, they lead to much bigger issues: user information leak, XSS and even one-click account takeovers.

Here are a few cases that I have encountered in the wild of CSRFs leading to severe security issues. Often, these are a combination of CSRF and other minor design flaws.

## Critical CSRF #1: Leaking user information using CSRF

CSRF sometimes causes information leaks as a side effect. Applications often send or disclose information according to user preferences. If these setting endpoints are not protected from CSRFs, they can pave the way to sensitive information disclosures. One way of achieving CSRF based info leaks is to play with these requests.

For example, a paid service on a web app that I have worked on sends monthly billing emails to a user-designated email address. These emails include the street addresses, phone numbers, and limited credit card information of the user. The email address to which these billing emails are sent can be changed via the following request:

```
POST /change_billing_email

REQUEST BODY:
email=NEW_EMAIL &csrftok=12345
```

The CSRF validation on this endpoint was broken. The server accepts a blank token and the request would succeed even if the *csrftok* field is left empty. After making a victim send the following request, all future billing emails would then be sent to ATTACKER_EMAIL (until the victim notices the unauthorized change), thereby leaking the street address and phone numbers associated with the account to the attacker.

```
POST /change_billing_email

REQUEST BODY:
email=ATTACKER_EMAIL &csrftok=
```

## Critical CSRF #2: Stored Self-XSS using CSRF

Self-XSS is almost always regarded by security teams as a non-issue because they are difficult to exploit. However, when combined with a CSRF, self-XSS can often be turned into a stored-XSS.

For example, on a financial site that I have come across, users are given the ability to create nicknames for each of their linked bank accounts. The account nickname field is vulnerable to self-XSS: there is no sanitization, validation or escaping for user input on the field. However, this is a field that only the authorized user can edit and see, so there is no way for an attacker to trigger the XSS directly.

Unfortunately, a CSRF bug also exists on the endpoint used to change the account nicknames. The application does not properly validate the existence of the CSRF token, so simply omitting the token parameter in the request will bypass CSRF protection. For example:

```
POST /change_account_nickname

REQUEST BODY:
nickname=<XSS PAYLOAD> &csrftok=WRONG_TOKEN
```

This request would fail.

```
POST /change_account_nickname

REQUEST BODY:
nickname=<XSS PAYLOAD>
```

While this request would successfully change the user's account nickname, and store the XSS payload. The next time a user logs into the account and view her dashboard, the XSS would be triggered.

## Critical CSRF #3: Taking over user accounts using CSRF

These are some of the easiest account takeovers that I have discovered. And these situations are not uncommon either. Account takeover issues occur when there is a CSRF issue in an account validating functionality like creating a password, changing the password, changing the email address, or resetting the password.

For example, here's a bug that I have discovered in a client's web app.

The web app allows social media sign-ups. And after a user signs up via social media, they have the option to set a password via the following request:

```
POST /password_change

REQUEST BODY:
oldpassword= &newpassword=XXXXX &csrftok=12345
```

Since the user signed up via their social media account, no old password is needed to set the new password. So if CSRF protection fails on this endpoint, an attacker would have the ability to set a password for anyone who signed up via their social media account and has not set a password.

And unfortunately, that's exactly what happened on this particular endpoint. The application does not validate the CSRF token properly and accepts an empty value as the *csrftok* parameter. So essentially, the following request will set the password of anyone (who doesn't already have a password set) to *ATTACKER_PASS*.

```
POST /password_change

REQUEST BODY:
oldpassword= &newpassword=ATTACKER_PASS &csrftok=
```

Now all an attacker has to do is to embed this request on pages frequented by users of the site, and she can automatically assign the password of any user who visits those pages to *ATTACKER_PASS*. After that, the attacker is free to log in as any victim with their newly assigned password.

## Conclusion

CSRFs are super common and very easy to exploit. While the majority of CSRFs that I have encountered proved to be low severity issues, sometimes an oversight on critical endpoints can lead to severe consequences.

If you are a developer, pay extra attention to the CSRF protection mechanisms deployed on critical endpoints.

If you are a hacker, think about what role the functionality plays in the context of the entire application when you encounter a CSRF. How does the endpoint affect the rest of the application? How could you escalate the vulnerability based on that knowledge?