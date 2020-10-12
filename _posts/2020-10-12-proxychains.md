---
title: "Proxying Like a Pro"
categories:
  - Hacking
---

Using ProxyChains to proxy your Internet traffic.

How do hackers cover their tracks during a cyber attack? Today, let's talk about an important concept for penetration testers and forensics investigators: proxying.

Proxying refers to the technique of bouncing your Internet traffic through multiple machines to hide the identity of the original machine, or to overcome network restrictions. ProxyChains is a tool that hackers often use to accomplish this goal.

## What is ProxyChains?

ProxyChains is a tool that forces any TCP connection made by any given application to go through proxies like TOR or any other SOCKS4, SOCKS5 or HTTP proxies. It is an open-source project for GNU/Linux systems.

Essentially, you can use ProxyChains to run any program through a proxy server. This will allow you to access the Internet from behind a restrictive firewall, hide your IP address, run applications like SSH/ telnet/wget/FTP and Nmap through proxy servers, and even access your local Intranet from outside through an external proxy.

ProxyChains even allows you to use multiple proxies at once by "chaining" the proxies together and to use programs with no built-in proxy support through a proxy.

One of the most important reasons that ProxyChains is used in a security context is that it's a trick to evade detection.

Attackers often use proxies to hide their true identities while executing an attack. And when multiple proxies are chained together, it becomes harder and harder for forensics professionals to trace the traffic back to the original machine. When these proxies are located across countries, investigators would have to obtain warranties in the local jurisdictions where every proxy is located. This makes the investigation very difficult.

## Setting Up ProxyChains

First off, let's go through how to install and configure ProxyChains! If you are using Mac OSX, you can simply install ProxyChains using Homebrew. Run the following command after you install Homebrew:

```bash
brew install proxychains
```

If you are using another Unix flavor, you have two options. First, if you are using a Debian-based Linux distribution, you can install ProxyChains with the command:

```bash
apt-get install proxychains
```

You can also build the project from source code. First, download the latest project source on ProxyChain's official release page [here](https://github.com/haad/proxychains/releases).

Then, "cd" into the project directory and run the following commands.

```bash
./configure
make
sudo make install
```

### Configuring ProxyChains

The proxying behavior of ProxyChains is fully customizable. You can, for example, choose from three different chaining options:

-   Strict Chain: all proxies in the list will be used and they will be chained in order.
-   Random Chain: each connection made through ProxyChains will be done via a random combo of proxies in the proxy list. Users can specify the number of proxies to use. (This option is useful for IDS testing.)
-   Dynamic Chain: the same as a strict chain, but dead proxies are excluded from the chain.

You can also specify the list of proxies that ProxyChain uses in the following format:

```bash
type host port [username password]
```

Here are some examples:

```bash
socks5 192.168.67.78 1080 lamer secret
http 192.168.89.3 8080 justu hidden
socks4 192.168.1.49 1080
```

The supported proxy types are socks4, socks5, and HTTP. You can list either a single proxy or multiple ones for ProxyChains to use.

You can see an example of a ProxyChains configuration file [here](https://github.com/haad/proxychains/blob/master/src/proxychains.conf).

ProxyChains looks for the configuration file in the following order:

-   SOCKS5 proxy port in environment variable ${PROXYCHAINS_SOCKS5},
-   file listed in environment variable ${PROXYCHAINS_CONF_FILE},
-   the -f argument provided to the proxychains command,
-   ./proxychains.conf,
-   $(HOME_DIRECTORY)/.proxychains/proxychains.conf,
-   and finally, /etc/proxychains.conf.

## Using ProxyChains

The most basic usage of ProxyChains is as follows. After configuring ProxyChains in the configuration file, you can simply run:

```bash
proxychains [original command]
```

To execute that command through ProxyChains. For example, this command will tunnel telnet through the listed proxies.

```bash
proxychains telnet targethost.com
```

While this command will simply connect to targethost.com through the listed proxies:

```bash
proxychains targethost.com
```

### Pivoting with ProxyChains

Pivoting is a technique that attackers use to reach machines that are protected from the Internet. Some machines are not directly reachable via the Internet but are reachable by other machines that are connected directly to the Internet. To attack these protected machines, attackers compromise the Internet-facing machine and use it to "pivot" into the Intranet.

![](https://vickieli.dev/assets/images/hacking-19.png)

For example, if an attacker cannot access an internal machine because of a firewall blocking traffic from outside the organization Intranet, she can find an Internet-facing machine in the Intranet (like a webserver), compromise that machine, and use the machine as a proxy to access the internal machine.

ProxyChains helps attackers do this. Attackers can find machines on the Intranet, set them as proxies in ProxyChain's proxy list, and pivot deeper and deeper into a network.

### Nmap via ProxyChains

You can also perform Nmap scans via ProxyChains.

```bash
proxychains [nmap command]
proxychains nmap -sT targethost.com
```

### ProxyChains via SSH

You might also want to use ProxyChains with SSH. To make ProxyChains work with SSH, you'll first need to configure SSH to work as a proxy. This can be done with the "-D" option for SSH.

```bash
ssh -D 127.0.0.1:8080 targethost.com
```

This will make SSH forward all traffic sent to port 8080 to targethost.com. You should then add `127.0.0.1:8080` to the ProxyChains proxy list.

## Conclusion

Proxying is an important skill to master for anyone working in infosec. Learning the mechanisms behind proxying and how to use proxies to achieve your goals will be very useful. And ProxyChains is a simple tool to help you proxy efficiently. Good luck!