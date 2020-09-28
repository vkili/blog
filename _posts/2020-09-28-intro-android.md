---
title: "An Android Hacking Primer"
categories:
  - Hacking
---

How to get started hacking Android applications.

Hi there. On this blog, I usually talk about web application vulnerabilities that I have seen in the wild. However, a while ago I started getting interested in hacking mobile applications, particularly, Android applications.

Android applications are becoming more important for many organizations. However, it is an often overlooked attack surface and, as a result, they are less well defended.

Hacking Android applications are for a large part, similar to hacking web applications, despite some minor differences and some special bug classes. In this blog post, we are going to talk about:

-   Getting started hacking Android applications,
-   The tools used to hack Android applications,
-   And the common bugs to look out for.

These are mostly a few things I learned after hacking some Android CTF challenges. Hopefully, these tips will provide you with a good foundation to get started in reversing and hacking Android applications. Here we go!

## Anatomy of an APK

In order to hack Android applications, we must first understand what they are actually made out of. What goes into an Android application that you install on your phone?

Android applications are distributed and installed in application package files (APKs). APKs are like ZIP files that contain everything an Android application needs to operate: the application code, the application manifest file, and the application's resources.

Here are the main components of an Android APK:

-   AndroidManifest.xml: contains the application's package name, access rights, referenced libraries as well as other metadata.
-   classes.dex: contains the application source code compiled in .dex file format.
-   resources.arsc: contains the application's precompiled resources.
-   res/ folder: contains the application's resources not compiled into resources.arsc.
-   lib/ folder: contains compiled code that is platform-dependent. Each subdirectory in lib/ contains the specific source code for respective processors.
-   assets/ folder: contains the application's assets.
-   META-INF/ folder: contains the MANIFEST.MF file, which stores metadata about the application. It also contains the certificate and signature of the APK.

## Tools to use

Now that we understand the main components of an APK, we'll need to know how to process the APK file before we jump into searching for vulnerabilities. Here are some tools that are essential to analyzing APK files.

I am not going to go into the specifics of how to use these tools, but rather, when and why use these tools. The rest you can easily figure out using the tool's documentation page!

### Android Debug Bridge ([Documentation here](https://developer.android.com/studio/command-line/adb))

The Android Debug Bridge, or adb, is a command-line tool that lets you communicate with a connected Android device. Adb allows you to push and pull files to your test device easily, and quickly install modified versions of the application you are hacking.

### Apktool ([Documentation here](https://ibotpeaches.github.io/Apktool/))

Apktool, AKA what I call "the Burp Suite of Android" is an essential tool for Android hacking. It will probably the most frequently used tool during your analysis.

Apktool is a tool for reverse engineering APK files. It lets you unpack APK files and repack them after making some modifications. This allows you to directly inspect the application's resources, and insert code into the application for further analysis.

### Android Studio

Android Studio is the standard environment for developing Android applications. You can use it to make modifications to the application, or run the application in a virtual environment via the emulator.

### Dex2jar ([Documentation here](https://github.com/pxb1988/dex2jar))

Dex2jar converts .dex files to .class files (zipped as Jar files). You can then decompile the Jar files for source code analysis.

### JD-GUI ([Documentation here](http://java-decompiler.github.io/))

JD-GUI is a decompiler for .class files. After running the APK through Dex2jar, you can use JD-GUI to display the application's Java code.

### Frida ([Documentation here](https://frida.re/))

Frida is an amazing instrumentation toolkit that lets you inject your own script into running processes of the application. This can be used to inspect functions that are called, the network connections of the application, and to bypass certificate pinning.

### Burp Suite

Lastly, we cannot forget about Burp Suite! Burp Suite can be used as a proxy to inspect the traffic to and from your test device. Documentation about setting up Burp to work with an Android device is [here](https://support.portswigger.net/customer/portal/articles/1841101-configuring-an-android-device-to-work-with-burp).

## Files to look into

After you have unpacked the APK file, you can immediately start looking into the files within the application to look for quick wins and the more obvious vulnerabilities. Here are a few files to look out for:

1.  AndroidManifest.xml: In this file, you can get basic information about the application and its functionalities.
2.  res/values/strings.xml: This file stores the string resources for the application. It's a good place to look for hardcoded secrets and info leaks.
3.  Files ending in .db or .sqlite: Look for these files to see what information is shipped along with the application. This is an easy source of secrets and info leaks.
4.  Java source code: A good place to look for other vulnerabilities, such as weak cryptography or insecure authentication methods.

## Flaws to look for

Hacking Android applications is very similar to hacking web applications. But there are specific classes of vulnerabilities that are more commonly present in Android apps. Here are a few of them:

### Hardcoded credentials

Android applications may contain hardcoded secrets or API keys for the application to access certain web services.

A good way to look for these is to unpack the APK file and grep for certain strings in the source files.

### Insecure data storage

Some applications will store sensitive data insecurely within the APK. Look for things like session data, financial information, and personal information.

A good way to test for this is to unpack the APK file and grep for certain strings.

### Web vulnerabilities

Android applications are an excellent place to search for additional web vulnerabilities that are not present in its web application equivalent. This is because mobile apps often make use of unique API endpoints that may not be as well tested as web API endpoints. Search for IDORs, SQL injections, file upload vulnerabilities, and other common web vulnerabilities.

A good way of finding these vulnerabilities is to use Burp Suite to intercept the traffic coming out of the mobile app during sensitive actions.

### Session management

In Android apps, sessions are often managed with a session token sent via a header. So an open redirect to a foreign server might lead to account takeover.

The same issues that plague session management in web apps, such as insufficient session expiration and reusing session tokens could also be an issue in Android apps.

### Weak cryptography

Some applications use custom implementations for encryption or hashing. Look for insecure algorithms, weak implementation of known algorithms and hardcoded encryption keys.