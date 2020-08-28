---
title: "Hacking Java Deserialization"
categories:
  - Insecure Deserialization
---

How attackers exploit Java Deserialization to achieve Remote Code Execution

Insecure deserialization bugs are often very critical vulnerabilities: an insecure deserialization bug will often result in arbitrary code execution, granting attackers a wide range of capabilities on the application.

Defending against deserialization vulnerabilities is also extremely difficult. How to protect an application against these vulnerabilities varies and depends greatly on the programming language, the libraries and the serialization format used. Because of this, there is no one-size-fits-all solution.

Because of these reasons, this class of vulnerabilities has always fascinated me. Today, let's learn to exploit deserialization vulnerabilities in Java applications!

## What is insecure deserialization?

To understand insecure deserialization, we must first understand what serialization is and how it is used in applications.

Serialization is a process during which an object in a programming language (say, a Java object) is converted into a format that can be saved to the database or transferred over a network. Whereas deserialization refers to the opposite: it's when the serialized object is read from a file or the network and converted back into an object.

Many programming languages support the serialization and deserialization of objects, including Java, PHP, Python, and Ruby.

Insecure deserialization is a type of vulnerability that arises when an attacker is able to manipulate the serialized object and cause unintended consequences in the program's flow. This can cause DoS, authentication bypass or even RCE.

For example, if an application takes a serialized object from the user and uses the data contained in it to determine who is logged in, a malicious user might be able to tamper with that object and authenticate as someone who she is not. If the application uses an unsafe deserialization operation, the malicious user might even be able to embed code snippets in the object and get it executed during deserialization! And this is what we'll focus on today: gaining arbitrary code execution using an insecure deserialization bug in a Java application.

## Serialization interface in Java

In order to understand how to exploit deserialization vulnerabilities, let's first quickly review how serialization and deserialization work in Java:

The serialization of Java classes is enabled by the class implementing the [java.io.Serializable](https://docs.oracle.com/en/java/javase/13/docs/api/java.base/java/io/Serializable.html) interface. Classes implement special methods: writeObject() and readObject() to handle the serialization and deserialization of objects of that class. Classes that do not implement this interface will not have any of their objects serialized or deserialized.

## Exploiting Java insecure deserialization

So how can we exploit Java applications via an insecure deserialization bug? The first step is to find an entry point to insert the malicious serialized object.

Serializable objects are often used in applications to transport data in HTTP headers, parameters or cookies in Java applications.

### The Java serialized object

Java serialized objects have the following signatures. These can help you recognize potential entry points for your exploits:

-   Starts with `AC ED 00 05` in Hex or `rO0` in Base64. (You might see this within HTTP requests as cookies or parameters.)
-   `Content-type` header of an HTTP response set to `application/x-java-serialized-object`.

Since Java serialized objects contain a lot of special characters, it is common to encode them before transmission. So look out for differently encoded versions of these signatures as well.

### Manipulating object data and application logic

After you discover a user-supplied serialized object, the first thing you can try is to manipulate program logic by tampering with the information stored within the objects.

For example, if the Java object is used as a cookie for access control, you can try changing the usernames, role names, and other identity markers that are present in the object and re-serialize it and relay it back to the application.

You can also try tampering with any sort of value in the object that is: a file path, file specifier, and control flow values to see if you can alter the program's flow.

### From bug to code execution

So what else can we do when an application deserializes uncontrolled user input? When the application does not put any restrictions on what classes are allowed to get deserialized, all serializable classes that the current classloader can load can get deserialized. This means that arbitrary objects of arbitrary classes can be created by the user! A potential attacker can achieve RCE by constructing objects of the right classes that can lead to arbitrary commands.

The path from a Java deserialization bug to remote code execution can be convoluted. To gain code execution, a series of gadgets need to be used to reach the desired method for code execution. This works very similarly to exploiting deserialization bugs using POP chains.

In Java applications, these gadgets are found in the libraries loaded by the application. Using gadgets that are in-scope of the application, you can create a chain of method invocations that eventually lead to RCE.

This chain can be executed either during or after the deserialization process. It is important that the first gadget is self-executing, so a common target for the first gadget would be a hook method during the deserialization process. You should also look for gadgets in commonly available libraries to maximize the chances that your gadgets are in-scope for the application.

Exploits have been developed and published utilizing gadgets in popular libraries such as the Commons-Collections, the Spring Framework, Groovy, and Apache Commons Fileupload.

### Limitations of this approach

There are, however, some limitations to this approach. First, it is very time consuming to find and chain gadgets to formulate an exploit. You are also limited to the classes that are available to the application, which can limit what you can do with the exploit.

In addition, gadget classes must implement serializable or externalizable, and different library versions may also yield different usable gadgets.

### Automating the exploitation: using Ysoserial

[Ysoserial](https://github.com/frohoff/ysoserial) is a tool that can be used to generate payloads that exploit Java insecure deserialization bugs, and save you tons of time developing gadget chains yourself.

Ysoserial uses a collection of gadget chains discovered in common Java libraries to formulate exploit objects. Using Ysoserial, you can create malicious Java serialized objects using gadget chains from specified libraries with a single command. The usage is as follows:

```bash
$ java -jar ysoserial.jar [gadget chain] '[command to execute]'
```

When an application with the required gadgets on in scope deserializes this object insecurely, the chain will automatically be invoked and cause the command to be executed on the application host. For example, to create a payload that uses a gadget chain in the Commons Collections library that would open a calculator on the target host you can use:

```bash
$ java -jar ysoserial.jar CommonsCollections1 calc.exe
```

Sometimes it would be obvious which library to use for your gadget chain, but often, it would be a matter of trial and error to see which vulnerable libraries are available to the application. (This is where good recon comes in!)

## Preventing Java insecure deserialization bugs

If you are dealing with the deserialization yourself, make sure not to deserialize any data tainted by user input without proper checks. If deserialization is absolutely necessary, restrict deserialization to a small list of allowed classes (use a whitelist).

It also helps to utilize simple data types, like strings and arrays instead of objects that need to be serialized on transport. To prevent the tampering of cookies, keep the session state on the server instead of relying on user input for session information.

Otherwise, keep an eye out for patches and keep dependencies up to date.

### What does not work

Some developers mitigate against deserialization vulnerabilities by identifying the commonly vulnerable classes and remove them from the application. This effectively restricts available gadgets.

However, this is not a reliable form of protection. It is necessary to address the root cause of this vulnerability: the insecure deserialization.

Limiting gadgets can be a great defense strategy but is not a cure-all for deserialization issues. Hackers are creative and can always find more gadgets in other libraries, and come up with creative ways to achieve the same results.
