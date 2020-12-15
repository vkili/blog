---
title: "Becoming Root Through Misconfigured SUDO"
categories:
  - System Security
---

Linux privilege escalation by exploiting SUDO rights.

Welcome back to the Linux Security Series! In this series, we'll discuss security issues that affect Linux systems and common misconfigurations that lead to them. Let's get started!

Privilege escalation is a way that attackers can escalate their privileges on a system. For example, let's say that an attacker has gained access to your web server, but only as a low-privileged user. They cannot read or write sensitive files, execute scripts, or change system configuration. How could they compromise your server and maintain their access there?

If they can find a way to trick the system into thinking that they are the root user, the attacker can carry out more powerful attacks like reading and writing sensitive files and inserting permanent backdoors into the system. And this is where privilege escalation comes in.

Today, let's talk about how attackers can exploit misconfigured SUDO rights to escalate their privileges.

## What Is SUDO?

SUDO stands for "Super User DO." It is a special permission that allows users to run programs with the security privileges of another user --- typically the root user.

Admins grant users SUDO access so that they can run certain commands as the root user. If the admin has granted SUDO rights for a command to a user, that user can run the command with root privileges by typing the `sudo` command before the actual command and then entering their password. For example, to run the `whoami` command as the root user, the user with SUDO rights can run this command:

```bash
$ whoami
vickieli
$ sudo whoami
root
```

Most of the time, admins will grant users SUDO rights to only a few commands. But even then, these SUDO rights can introduce privilege escalation issues.

## SUDO Privilege Escalation

So how can attackers exploit their SUDO rights to execute arbitrary commands as the root user?

If the attacker has SUDO rights to programs that allow command execution or arbitrary writes to files on the system, the attacker can exploit the temporary root access to execute code as root on the system. You can find the commands the current user can run with SUDO via this command:

```bash
$ sudo -l
```

For example, let's say that regular users are given the ability to run the command `find` with SUDO so that they can search for all files on the system. The `find` command is usually used for locating files and often has SUDO permissions to allow users to find files across the system. But `find` allows the execution of system commands through the `-exec` flag! For example, to run the `ls` command from within the `find` command, you can use the command `find . -exec ls \;`. So if a user can run the `find` command as SUDO, they can execute system commands as root!

These misconfigurations make privilege escalation trivial. For example, an attacker can use the ability to execute commands as root and add themselves as a root user in the `/etc/passwd` file. This command will do just that:

```bash
$ find . -exec echo "vickie::0:0:System Administrator:/root/root:/bin/bash" >> /etc/passwd \;
```

This command adds a root user with the username `vickie` and an empty password. Since `0` is the UID of the root user, adding a user with the UID of `0` will give that user root privileges. This command is not possible for regular users because only privileged users can modify system-critical files such as the `/etc/password` file.

Many programs give users the ability to execute system commands, and `find` is just one of them. Other examples include `vim`, `python`, `less`, and `more`.

## More SUDO Dangers

Programs that lead to privilege escalation when run with SUDO are not just limited to programs that allow for arbitrary system code execution. Any programs that allow arbitrary writes to system files can lead to privilege escalation when run with SUDO.

For example, if the file editor Nano can be run with SUDO, the attacker can use Nano's root permissions to open the `/etc/passwd` file and add themselves as the root user directly in the file editor!

And the system utility cp is used to copy and overwrite files. If it can be run with SUDO, attackers can tamper with any file on the system by overwriting the original file with its root privileges! For example, the attacker can create a copy of the original `/etc/password` to a file they own. Then, they can add themselves as a root user by editing the copy of the `passwd` file. Finally, they use the `cp` command to overwrite the original `/etc/password` file with the modified one.

## Be Careful!

SUDO rights should be granted carefully. Sure, some programs need it because they need root privileges to function properly. But if you grant SUDO rights to programs that allow users to execute system commands or write to random system files, you introduce privilege escalation vulnerabilities into your system.

Thanks for reading! Next time, we'll dive into more privilege escalation techniques that attackers can use to compromise your system.