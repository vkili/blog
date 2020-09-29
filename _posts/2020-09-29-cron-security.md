---
title: "Privilege Escalation Via Cron"
categories:
  - System Security
---

How attackers can exploit misconfigured Cron permissions to gain root access.

Cron is a super useful job scheduler in Unix-based operating systems. It allows you to schedule jobs to run periodically.

Cron is usually used to automate system administration tasks. But for the individual user, you can use Cron to automate tasks like downloading emails, running malware scanners and checking websites for updates.

Today, let's dive into how to use Cron and the security risks of a misconfigured Cron system.

## How Does Cron Work?

The behavior of the Cron utility can be fully customized. You can configure the behavior of Cron by editing files called "crontabs". Unix keeps different copies of crontabs for each user. You can edit your own user's crontab by running:

```bash
crontab -e
```

You can also list the current cronjobs for your user by running:

```bash
crontab -l
```

There is also a system-wide crontab that administrators can use to configure system-wide jobs. In Linux systems, the location for the system-wide crontab is `/etc/crontab`. Cron will run as the root user when executing scripts and commands in this file.

### Crontab syntax

All crontabs follow the same syntax. Each line specifies a command to be run and the time at which it should run.

```
* * * * * <command to be executed>
- - - - -
| | | | |
| | | | ----- Weekday (0 - 7) (Sunday is 0 or 7, Monday is 1...)
| | | ------- Month (1 - 12)
| | --------- Day (1 - 31)
| ----------- Hour (0 - 23)
------------- Minute (0 - 59)
```

For example, this crontab entry tells the system to `cd` into the directory where I store security scripts and run the `scan.sh` shell script every day at 9:30 pm. (The wildcard character "\*" means "all".)

```
30 21 * * * cd /Users/vickie/scripts/security; ./scan.sh
```

And in system-wide crontabs, you can also specify the user to run the command as:

```
* * * * * <username> <command to be executed>
```

For example, this entry will tell Cron to run the same commands, but as the root user:

```
30 21 * * * root cd /Users/vickie/scripts/security; ./scan.sh
```

### Running scripts in batches

It is customary to place scripts that the system-wide crontab uses in the `/etc/cron.d`, `/etc/cron.hourly`, `/etc/cron.daily`, `/etc/cron.weekly` and `/etc/cron.monthly` directories.

You can then batch run the scripts within the directories. For example, the following line in the crontab tells Cron to run all scripts in the `/etc/cron.hourly` directory as root every hour.

```
01 * * * * root run-parts /etc/cron.hourly
```

## Cron Privilege Escalation

So how does Cron become a source of vulnerabilities?

By default, Cron runs as root when executing `/etc/crontab`, so any commands or scripts that are called by the crontab will also run as root. When a script executed by Cron is editable by unprivileged users, those unprivileged users can escalate their privilege by editing this script, and waiting for it to be executed by Cron under root privileges.

Let's say the following line is in `/etc/crontab`. Every day at 9:30 pm, Cron runs the `maintenance.sh` shell script. Since the script is called from `/etc/crontab`, it will run under root privileges.

```
30 21 * * * cd /path/to/maintenance.sh
```

Now let's say that the `maintenance.sh` script is also editable by everyone, not just the root user. In this case, anyone can add commands to `maintenance.sh`, and get that command executed by the root user.

This makes privilege escalation trivial. For example, attackers can grant themselves Superuser privileges by adding themselves as a Sudoer.

```bash
echo "vickie ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

Or, they can gain root access by adding a new root user to the `/etc/passwd` file. In this command below, "0" is the UID of the root user, so adding a user with the UID of "0" will give that user root privileges. This user will have the username of "vickie" and an empty password:

```bash
echo "vickie::0:0:System Administrator:/root/root:/bin/bash" >> /etc/passwd
```

And so on. There are many more ways to escalate a user's privilege on a Unix-based system. By exploiting a misconfiguration in a crontab, the attacker will be able to execute any command of their choosing and gain root privileges.

### What if my file permissions are secure?

Another common security hole is vulnerabilities in the scripts themselves. If a script behaves in an insecure manner and you run it as root using Cron, then that could introduce vulnerabilities too.

For example, let's say that the system-wide crontab runs a script that contains a wildcard injection vulnerability.

A wildcard injection vulnerability happens when a program uses the wildcard (\*) character in an insecure way. This allows attackers to change the command's behavior by injecting command flags. In this case, the vulnerability occurs within these lines in the script:

```bash
cd directory1chown root *
```

The script goes into `directory1` and changes the owner of every file to the root user. If the directory contains the files `a.txt`, `b.txt`, and `c.txt`, the second command would expand into the following, due to the wildcard.

```bash
chown root a.txt b.txt c.txt
```

But the command chown has a flag called `--reference`, which tells chown to change the owner of the files to the owner of the reference file instead. So this command would change the owner of all files to the user "vickie" instead.

```bash
chown root a.txt b.txt c.txt --reference=file_owned_by_vickie.txt
```

So how can a hacker exploit this situation?

First, she can create a file in `directory1` using her own user account, called `file_owned_by_vickie.txt`. Then, she can create another file in `directory1` called `--reference=file_owned_by_vickie.txt`.

Finally, when the script gets executed, the wildcard will notice that the directory contains five files: `a.txt`, `b.txt`, `c.txt`, `--reference=file_owned_by_vickie.txt` and `file_owned_by_vickie.txt`. It will expand the command into this one:

```bash
chown root a.txt b.txt c.txt --reference=file_owned_by_vickie.txt file_owned_by_vickie.txt
```

Our `--reference` flag was injected and thus, the owner of the `file_owned_by_vickie.txt` file now owns all files in `directory1`.

Normally, chown is executable only by a superuser, but running it through Cron as the root account gives attackers the opportunity to exploit the wildcard injection vulnerability.

## Conclusion

If your system uses Cron to automate tasks, make sure that none of the scripts that you run through crontab are editable by unprivileged users, and make sure that your Cron scripts are secure! You could accidentally leave your system wide open to privilege escalation attacks.