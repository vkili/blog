---
title: "A Deep Dive Into Linux Permissions"
categories:
  - System Security
---

Learn about the Linux permission model and how it affects your system's security.

Welcome to the Linux Security Series! In this series, I will tackle the fundamentals of Linux security and how attackers attack Linux machines.

Before we dive into Linux systems security, it's essential to understand the permissions model of Linux machines. Understanding how to manage permissions on Linux systems will help you understand attacks that exploit the permissions system, like most privilege escalation techniques. Let's get started.

## Linux File Permissions

Linux inherited the Unix model of file ownership and permissions. Every file and folder on the system has a set of *permissions* that specifies who is allowed to do what with that particular file.

There are three types of permissions: *read*, *write*, and *execute*. A *read* permission on a file enables a user to read the contents of the file. A *write *permission allows a user to modify or delete the file. And an *execute *permission allows a user to run the file as a script or an executable.

You can view the permissions of a file or directory by using the `ls -l` command in a directory. You should see a line like this.

```bash
-rwxrwxrwx
drwxrwxrwx
```

The first character indicates whether the item is a file or a directory. A dash means that the item is a file, whereas a *d* means it's a directory. The next three characters are the permissions of the file's owner. The owner is usually the user who created the file and has the most control over it. *R* indicates *read*, *w* indicates *write*, and *X* indicates *eXecute*. And a dash indicates the lack of that permission. Let's look at an example. Here, the owner can read, write, and execute the file.

```bash
-rwxr--r--
```

And in Linux, users are sorted into user *groups*, and these groups often share file permissions. The next three characters are the permissions of the owner's group. And the final three are the permissions for everyone else. This file's permissions indicate that it is readable by everyone but only the owner can write or execute the file.

You can set file permissions by using the `chmod` command. In this command, you use the characters *u*, *g*, and *o* to indicate the *owner user*, *owner group*, and *others*. For example, to add execution permissions for a file's owner group, you can use the command:

```bash
chmod g+x filepath
```

And to set execute permissions for everyone, you can use the command:

```bash
chmod +x filepath
```

On the other hand, when you want to remove a permission, you can swap out the plus sign for a minus sign. For example, this command will remove execute permissions for everyone.

```bash
chmod -x filepath
```

## Special Permission Modes

There are also a few special things you can do with a file's permissions.

The first thing you can do is set the setUID, or SUID bit. When the `SUID` bit is set, the file will always run as the user who owns the file and not as the user who started the program. For example, if an executable is owned by root, then the file will always run as the root user, regardless of who started the execution. You can tell if a file has SUID permissions if there is an *s* character instead of an *x* or a dash in the owner's permissions.

```bash
-rwsrwxrwx
```

The SUID bit has many common use cases. For example, since the `ping` command needs to be executed with root privileges, if you want normal users to use `ping`, you need to set the SUID bit for the `ping` executable. The SUID bit can be set by using the `chmod` command as well. This command will set the SUID bit on a file.

```bash
chmod u+s filepath
```

SetGID, or SGID, works similarly to SUID. When the setGID bit is set on a file, all users can execute the file with the owner group's permissions. And if SGID is set on a directory, all the files created in that directory become accessible to all users in the parent directory's owner group. You can tell if a file or directory has SGID permissions if there is an *s* character instead of an *x* or a dash in the owner group's permissions.

```bash
-rwxrwsrwx
```

The SGID bit can be set by using `chmod` as well. This command will set the SGID bit on a file.

```bash
chmod g+s filepath
```

There is also something called the *sticky bit* in a file's permissions. If the sticky bit is set on a regular file, it makes subsequent execution of the program faster. However, the sticky bit is more commonly used on directories. It would mean the files or directories within that directory can only be moved or deleted by the file or directory's owner or the superuser. This is commonly used for the temp directory (`/tmp`), which is designed to store temporary files created by individual users. You can tell if a file or directory has the sticky bit set by seeing if there is a *t* character instead of an *x* or a dash in the permissions string's last character.

```bash
-rwxrwxrwt
```

And you can add the sticky bit to a file or directory by using this command.

```bash
chmod +t filepath
```s

I hope this helps you understand more clearly how Linux permissions are managed. Next time, we'll dive into some privilege escalation techniques that allow attackers to access or execute files despite not having permission. See you next time!