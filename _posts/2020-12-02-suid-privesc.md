---
title: "Becoming Root Through An SUID Executable"
categories:
  - System Security
---

Linux privilege escalation by exploiting the SUID bit.

Welcome back to the Linux Security Series! In this series, we'll discuss security issues that affect Linux systems and common misconfigurations that lead to them. Let's get started!

Privilege escalation is a way that attackers can escalate their privileges on a system. For example, let's say that an attacker has gained access to your web server, but only as a low privileged user. They cannot read or write sensitive files, execute scripts, or change system configuration. How could they compromise your server and maintain their access there?

If attackers can find a way to trick the system into thinking that they are the root user, they can carry out more powerful attacks like reading and writing sensitive files and inserting permanent backdoors into the system. And this is where privilege escalation comes in. Today, let's talk about how attackers can exploit SUID programs to escalate their privileges to become root.

## The SUID Bit

SUID stands for "SetUID". It is a Linux permissions flag that allows users to run that particular executable as the executable's owner. For example, if a file is owned by root, the program will always run as root, regardless of who started the execution.

Why would this functionality be useful? A common use-case for SetUID is the password change utility. To change your own password, you would have to modify sensitive system files, such as `/etc/shadow`. This file is normally only accessible by root users, so you will need to have root privileges to carry out a password change. But since the system doesn't want to give a normal user root privileges, SUID allows users to obtain root privileges only when running certain programs. In this case, SUID on the password change utility allows users to gain temporary root access to change their passwords without obtaining root access across the board. For the most part, this is a normal and necessary behavior. But if an attacker can find a way to execute arbitrary code when running these SUID programs, they can exploit the temporary root access to execute code as the root user on the system!

For example, let's look at the "Vim" file editor first. Let's say that Vim is owned by the root user and has the SUID bit set on a system. This means that whenever a user runs the Vim editor, Vim is running with root privileges. No biggie, right?

This setting could actually spell the death sentence for your system because you can actually run arbitrary system commands from within the vim editor! To run commands in Vim, you have to type the characters colon bang ":!", then the command you want to run. For example, to run the "ls" command, you can type "`:!ls`" then press enter. You should now see the results of the ls command on your terminal.

So this means that if the Vim executable has the SUID bit set, the attacker can execute system commands as root from the Vim editor! Other editors, such as "more" and "less" also allows for command execution from within the program. You can type the bang character "!" then the command that you want to execute for these programs. For example, to run the "ls" command, you can type "`!ls`" then press enter. You should now see the results of the ls command on your terminal.

Besides file editors, another program that allows users to run arbitrary system commands is the "find" command. The find command is usually used for locating files and often has the SUID bit set to allow users to find files across the system. But find allows the execution of system commands through the "`-exec`" flag! For example, to run the "ls" command from within the find command, you can use the command "`find . -exec ls \;`". So if the find executable has the SUID bit set, the attacker can execute system commands as root!

## Escalating Privileges Using The Vulnerability

These misconfigurations make privilege escalation trivial. For example, an attacker can use the ability to execute commands as root and add themselves as a root user in the `/etc/passwd` file. This command will do just that.

```bash
echo "vickie::0:0:System Administrator:/root/root:/bin/bash" >> /etc/passwd
```

This command adds a root user with the username of "vickie" and an empty password. Since "0" is the UID of the root user, adding a user with the UID of "0" will give that user root privileges. This command is not possible for regular users because only privileged users can modify system-critical files such as the `/etc/password` file.

## More SUID Dangers

Programs that lead to privilege escalation when run with SUID are not just limited to programs that allow for arbitrary system code execution. Any programs that allow arbitrary writes to system files are owned by root and have the SUID bit set can lead to privilege escalation.

For example, if the file editor "Nano" has the SUID bit set, the attacker can use Nano's root permissions to open the "`/etc/passwd`" file and add themselves as the root user directly in the file editor!

And the system utility "cp" is used to copy and overwrite files. If it has the SUID bit set, attackers can tamper with any file on the system by overwriting the original file with its root privileges! For example, the attacker can create a copy of the original `/etc/password` to a file they own. Then, they can add themselves as a root user by editing the copy of the `passwd` file. Finally, they use the "cp" command to overwrite the original`/etc/password` file with the modified one.

## Be Careful!

You can see that SUID could become incredibly dangerous when misused. SUID rights should only be granted to programs when necessary and not to programs that allow command execution or arbitrary writes to files on the system.

Thanks for reading! Next time, we'll dive into more privilege escalation techniques that attackers can use to compromise your system.