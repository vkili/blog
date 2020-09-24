---
categories:
  - Binary Exploitation
---

And how the Heartbleed bug works!

Welcome back to the binary exploitation series! This is the series where we learn about binary exploitation, the theory underlying binary exploitation techniques, and the defense mechanisms developed against them.

Today, let's talk about a class of vulnerabilities that is quite similar to buffer overflows: buffer overreads. By the way, be sure to familiarize yourself about buffer overflows [here](https://vickieli.dev/binary%20exploitation/buffer-overflow/).

## What is a buffer overread?

A buffer overread is like a buffer overflow, except that it occurs during a read operation. While reading from a buffer, the program goes over the buffer boundary and reads adjacent memory.

```
* Program is suppose to read til here * But reads til here...
<-------------------------------------><--------------------------->
     Buffer                                Other program data
```

In languages like C and C++, programs are free to access data in any part of the virtual memory via a pointer. Because of this, buffer overread issues can occur when pointers or their indexes are incremented beyond the bounds of the buffer (when iterating an array or reading a string), or when pointer arithmetics yields a result outside a valid memory address.

This could sometimes be leveraged by attackers to leak confidential information.

## How Heartbleed works

Most of you have probably already heard of Heartbleed (CVE-2014--0160). Heartbleed was a security bug found in the OpenSSL cryptography library and disclosed back in 2014. The vulnerability led to widespread exploitation and the theft of financial and medical information of millions.

The root cause of this vulnerability is a buffer overread bug in OpenSSL's implementation of the [TLS/DTLS heartbeat extension](https://tools.ietf.org/html/rfc6520), which is used to test that the secure communication links are alive. The protocol works like this:

1.  Machine1 sends the server a heartbeat request message. This message includes a payload (an arbitrary string) and a payload_length (the length of said payload).
2.  Machine2 responds with a heartbeat response message. It includes the same payload string that is received from the sender.
3.  Machine1 receives the response and verify that the response message contains the expected payload string.

### So what was the bug?

The Heartbleed bug occurred due to the faulty implementation of step 2. The vulnerable version of OpenSSL allocates a buffer to contain the return payload based on the value of payload_length, without checking if the actual length of the payload is equal to payload_length.

An attacker could, therefore, leak information from the target machine by sending a heartbeat request message with a payload_length larger than the actual length of the payload. The target machine would then return data of adjacent memory locations up to the size payload_length. The exploitation steps look like this:

1.  The attacker's machine sends the heartbeat request with a ten-byte payload and a payload_size of 1000 bytes.
2.  The victim server responds with the ten-byte payload, along with 990 bytes of data from adjacent memory locations.

```
* Program reads payload_length (1000 bytes) from memory ......
<--------------------------------------><-------------------------->
  Payload (10 bytes)                      Other program data
```

## Conclusion

I hope this post was helpful in clarifying what buffer overreads are. Buffer overreads are simple yet powerful and can lead to severe consequences.

For developers, it is extremely important to implement checks to verify that the program does not read and output info beyond the allocated buffer.

And that's it for today! Thanks for reading and see ya next time!