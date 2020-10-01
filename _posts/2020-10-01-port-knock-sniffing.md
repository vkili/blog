---
title: "Stealing Port Knock Sequences For Server Access"
categories:
  - System Security
---

How the incorrect use of port knocking can lead to system compromise.

Port scanning is a method that attackers use to gain knowledge about a system. Hackers use port scanning to enumerate services on a system and reveal any potentially exploitable services. After that, they can try to exploit these services for illicit access to the system.

So even if port scanning is not a vulnerability by itself, it is a good defense-in-depth strategy to prevent people from doing so. Systems can prevent port scanning by using "port knocks". Today, we talk about the port knocking technique, how not to use it, and a method hackers use to bypass this protection.

## What Is Port Knocking?

Port knocking is a way to externally open ports that are closed. External clients perform a "knock" by sending a secret sequence of packets to predetermined ports on the destination machine. These "knocks" can consist of anything from a single packet, to a number of encrypted packets. On the server, a daemon monitors the firewall log files for these special knock sequences and allows connections to ports accordingly.

The purpose of implementing port knocking is to protect the system against port scanning and enumeration of potentially vulnerable services. Without a valid port knock, the protected ports will appear unavailable to the attacker, thus hiding the services of a system.

### Incorrect Use Of Port Knocking

Since port knocking is kind of like a "password" to a port or service, it can be used to authenticate users to the server, right?

Port knocking is a good defense-in-depth strategy, but it should never be used as an authentication method for a server. Port knocking essentially protects a system via security through obscurity: it relies on the secrecy of the knock sequence. If the knock sequence is ever revealed to untrusted parties, authentication on all machines using the knock sequence will be compromised.

There are a few ways that a knock sequence can potentially be leaked. The knock sequence could be accidentally published. It could also be leaked through compromised log files. Finally, it could be stolen through packet sniffing if the knock sequence is unencrypted.

## Stealing Port Knocks

So how does an attacker go about stealing port knocks via packet sniffing?

First off, she can use Wireshark to listen in on traffic between other machines and the target server. Wireshark is a network protocol analyzer that allows you to capture packets on the network.

[Wireshark](https://www.wireshark.org/)

After collecting a network trace of suitable length from the network, she can filter out the traffic between legitimate client machines and the target server. For example, in the screenshot below the attacker uses IP address as the criteria to isolate the relevant packets.

![](https://vickieli.dev/assets/images/hacking-10.png)

She can then deduct the correct knock sequence from these packets and gain unauthorized access to a server and the attached network. Note that it does not matter how long or complicated the port knock sequence is. If the sequence is unencrypted, it is easy to steal.

### Preventing port knock security holes

Port knocks should not be relied on as a primary authentication method. It should instead be used in conjunction with another mechanism that is not vulnerable to man-in-the-middle attacks. And above all, sensitive network traffic like port knocks should always be encrypted!

## Conclusion

Port knocks are an extra layer of security that could be implemented to prevent port scanning. However, security measures are only effective if they are built correctly. Follow best practices when designing port knocks and always keep in mind the possibility of port knock sniffing!