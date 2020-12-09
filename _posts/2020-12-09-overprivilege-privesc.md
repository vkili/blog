---
title: "Becoming Root Through Overprivileged Processes"
categories:
  - System Security
---

Linux privilege escalation by exploiting an overprivileged process.

Welcome back to the Linux Security Series! In this series, we'll discuss security issues that affect Linux systems and common misconfigurations that lead to them. Let's get started!

Privilege escalation is a way that attackers can escalate their privileges on a system. For example, let's say that an attacker has gained access to your web server, but only as a low-privileged user. They cannot read or write sensitive files, execute scripts, or change system configuration. How could they compromise your server and maintain their access there?

If they can find a way to trick the system into thinking that they are the root user, the attacker can carry out more powerful attacks like reading and writing sensitive files and inserting permanent backdoors into the system. And this is where privilege escalation comes in.

Today, let's talk about how attackers can exploit an overprivileged process to escalate their privileges.

## What Is an Overprivileged Process?

An overprivileged process is a process that is running with more permissions than it requires. It is a security risk because if an attacker hijacks the application that runs with high privilege, the attacker can gain its permissions on the system. Today, let's look at things that attackers can do when they encounter an overprivileged process running as root.

Let's say that a web application suffers from a classic command injection attack. The application allows users to read a file by submitting the filename via a GET request parameter:

```
https://example.com/read?filename=abc.txt
```

The application's PHP source code looks like this. The application retrieves the URL parameter from the user and then concatenates it directly into a system command:

```php
<?php 
  $file=$_GET['filename']; 
  system("echo $file");
?>
```

The application lacks any input validation on the "system" call and enables attackers to execute arbitrary system commands via command injection. For example, the attacker can execute system commands by injecting it into the filename parameter:

```
https://example.com/read?filename=abc.txt;ls
```

This will cause the application to execute this command, which will print the contents of `abc.txt` and then list the contents of the current directory:

```bash
echo abc.txt;ls
```

But what if the web application has root privileges? Then the attacker can do a lot worse because the injected command will also run under root privileges. For example, the attacker can use the command injection to add themselves as a root user by editing the `/etc/passwd` file:

```
https://example.com/read?filename=abc.txt;echo+"vickie::0:0:System Administrator:/root/root:/bin/bash">> /etc/passwd
```

This command adds a new root user to the `/etc/passwd` file:

```bash
echo "vickie::0:0:System Administrator:/root/root:/bin/bash" >> /etc/passwd
```

Since `0` is the UID of the root user, adding a user with the UID of `0` will give that user root privileges. This user will have the username `vickie` and an empty password. This command is normally not possible for normal users because only privileged users can modify the `/etc/password` file. But since the web application runs as root, the command succeeds and the attacker gains root access to the system.

Overprivileged processes are not only a danger when attackers can execute arbitrary code. Let's say that a path traversal attack exists on this endpoint in a web application as well:

```
https://example.com/read?filename=abc.txt
```

An attacker can read files outside of the current directory by using the sequence `../` in the filename parameter to escape the current directory:

```
https://example.com/read?filename=../../../../etc/shadow
```

The `/etc/shadow` file is a file in Linux systems that contains the hashed passwords of system users. It is only readable by privileged users. If the web application has the permissions to view the `/etc/shadow` file, an attacker can utilize the path traversal vulnerability to read this file. The attacker can then crack the passwords they found in this file to gain access to privileged users' accounts on the system.

## Be Safe!

The attacks I discussed in this article are all caused by applications running with too much privilege. To prevent these issues, you should implement the *Principle of Least Privilege*.

This principle means that applications and processes should only be granted the privileges they need to complete their tasks. For example, when an application requires only read access to a file, it should not be granted any write or execute permissions. You should always check if user-facing applications like web servers and file servers are running as root. And you should never use "run as root" as the default solution to permission issues. You can grant applications precisely the permissions that they need instead. This will lower your risks of complete system compromise during an attack.

Thanks for reading! Next time, we'll dive into more privilege escalation techniques that attackers can use to compromise your system.
