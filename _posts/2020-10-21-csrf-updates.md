---
title: "Are CSRFs Still a Thing?"
categories:
  - CSRF
---

What SameSite by default means for the future of CSRFs.

CSRF vulnerabilities happen when attackers can initiate forged state-changing requests from a foreign domain. This usually occurs because the user's browser sends session cookies regardless of where the request originates from.

Besides implementing CSRF tokens to ensure the authenticity of requests, another way of protecting against CSRF is `SameSite` cookies.

## SameSite Cookies

A web application instructs the user's browser to set cookies via a `Set-Cookie` header. For example, this header will make the client browser set the value of the cookie `PHPSESSID` to `UEhQU0VTU0lE`:

```
Set-Cookie: PHPSESSID=UEhQU0VTU0lE
```

Besides the basic "cookie_name=cookie_value" designation, the `Set-Cookie` header allows several optional flags you can use to protect your users' cookies. One of them is the `SameSite` flag, which helps prevent CSRF attacks. When the `SameSite` flag on a cookie is set to `Strict`, the client's browser will not send the cookie during cross-site requests.

```
Set-Cookie: PHPSESSID=UEhQU0VTU0lE; Max-Age=86400; Secure; HttpOnly; SameSite=Strict
```

Another possible setting for the `SameSite` flag is `Lax`. This setting tells the client's browser to send a cookie only in `GET` requests that cause top-level navigation. This setting ensures that users still have access to the resources on your site if the cross-site request is intentional.

For example, if you navigate to Facebook from a third-party site, your Facebook logins would be sent. But if a third-party site initiates a `POST` request to Facebook or tries to embed the contents of Facebook within an Iframe, cookies would not be sent.

```
Set-Cookie: PHPSESSID=UEhQU0VTU0lE; Max-Age=86400; Secure; HttpOnly; SameSite=Lax
```

Specifying the `SameSite` attribute is good protection against CSRF because both the `Strict` and `Lax` settings will prevent browsers from sending cookies on cross-site form `POST`, `AJAX` requests, and within iframes and image tags. This renders the classic CSRF hidden form attack useless.

## SameSite by Default

Earlier this year, Chrome and a few other browsers made `SameSite=Lax` the default cookie setting if it's not explicitly set by the web application. This means that even if a web application does not implement CSRF protection, attackers will not be able to attack a victim using the Chrome browser using POST CSRF.

So in the future, the efficacy of a classic CSRF attack will be greatly reduced since Chrome has the largest web browser market share.

On Firefox, the `SameSite by default` setting is a feature that needs to be enabled. You can enable it by going to `about:config` and setting `network.cookie.sameSite.laxByDefault` to `true`.

## Is CSRF Still Possible?

Yes. Even with browsers adopting the `SameSite by default` policy, CSRFs are still possible under some conditions.

First, if the site allows state-changing requests with the `GET` HTTP method, then third-party sites can attack users by creating CSRF with a `GET` request.

For example, if the site allows you to change a password with a `GET` request, attackers could embed a link like this in forums to trick users into clicking on it:

<https://email.example.com/password_change?new_password=abc123>

In this case, since clicking on the link will cause top-level navigation, the user's session cookies will be included in the `GET` request and the CSRF attack will succeed.

```
GET /password_change?new_password=abc123
Host: email.example.com
Cookie: session_cookie=YOUR_SESSION_COOKIE
```

Another scenario is when sites manually set the `SameSite` attribute of a cookie to `None`. Some web applications have features that require third-party sites to send cross-site, authenticated requests. In that case, developers might explicitly set `SameSite` on a session cookie to `None`. When the `SameSite` attribute is set to `None`, sending the cookie cross-site is allowed, so traditional CSRF attacks would still work.

Finally, if the victim is using a browser that does not set the `SameSite` attribute to `Lax` by default (like IE and Safari), traditional CSRF attacks would still work if the target application does not implement diligent CSRF protection.