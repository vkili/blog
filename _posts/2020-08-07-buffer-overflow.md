---
categories:
  - Binary Exploitation
---

And how a jammed laptop key led to code execution?!!

Hi there! Welcome back to the binary exploitation series! In the coming posts, we are going to explore concepts and tricks used in binary exploitation.

## What is memory corruption?

Memory corruption refers to an attacker modifying a program's memory to her will, in a way that was not intended by the program. Through corrupting program memory, an attacker can make the program misbehave: she can potentially make the program leak sensitive info, execute her own code, or make the program crash. Most real-world system-level exploits involve some sort of memory corruption.

In this post, we're gonna dive into the bread and butter of memory corruption techniques: buffer overflows.

## What is a buffer overflow?

Buffers are areas of memory that are meant to hold data. For example, when a program accepts user input to later operate on, a chunk of memory would have to be set aside to store that user input.

Buffer overflow refers to when a program writes data to a buffer, the data takes up more space than the memory allocated for the buffer, thus causing the data to overwrite adjacent memory locations.

Before the buffer overflow happens, the memory allocation looks like this:

```
<---------------------------> <------------------------------------>\
         Buffer                     Other program data
```

If the input size does not exceed the buffer, all is good:

```
AAAAAAAAAA                    BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB\
<---------------------------> <------------------------------------>\
         Buffer                     Other program data
```

But when the user input size exceeds the size of the buffer, user input could overwrite other potentially important program data:

```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBBBBBBBBBBB\
<---------------------------> <------------------------------------>\
         Buffer                     Other program data
```

### Stack buffer overflow vs heap buffer overflows

There are two main types of buffer overflows: stack overflows and heap overflows.

Stack overflows corrupt memory on the stack. This means that values of local variables, function arguments, and return addresses are affected.

Whereas heap overflows refer to overflows that corrupt memory located on the heap. Global variables and other program data are affected.

## What are the potential consequences?

What an attacker would be able to do with a buffer overflow depends on where the buffer is located and what protections are in place.

### Redirect program flow

In both stack and heap overflows, attackers can overwrite important control variables in the program to redirect program flow. For example, attackers can overwrite the secret values used for authentication and reach restricted areas in the application.

It might also be possible for attackers to redirect program flow by overwriting certain function pointers and exception handlers.

### Code execution

During a stack overflow attack, if an attacker is able to place crafted code in memory, she can then overwrite the return address on the stack to point to the location of her malicious code. This way, the attacker can redirect program execution to her code snippet after the current function returns.

### Denial of service

Even if the attacker is unable to redirect the program flow in a meaningful way or gain the ability to execute code, overflows can result in the corruption of program data and thus lead to a crash.

## Preventing buffer overflow

Multiple techniques have been developed to mitigate the risk of buffer overflows. Details of these techniques are beyond the scope of this post, but if you are interested in them, we will go into them sometime later.

### OS level defenses

#### Stack Canaries

We'll get into them in later posts, but here's the digest version of what canaries are. Stack canaries are random values placed in memory just before the return address.

In order to overwrite the return address and redirect program flow, an attacker would have to overwrite the stack canary as well. And thus the program would be able to detect stack overflow by checking if the canary value is correct.

Before buffer overflow occurs, the canary value is RANDOM NUMBER:

```
 (RAMDOM NUMBER) 0xDEADBEEF\
<---------------------------> <-------------> <-------------------->\
         Buffer                  Canary         Return address
```

After buffer overflow, the canary value is altered:

```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA(ATTACKER CODE)\
<---------------------------> <-------------> <-------------------->\
         Buffer                  Canary         Return address
```

At this point, the system detects an attack and would not redirect to the attacker-controlled address.

This makes exploitation a lot harder, but canary protection can be bypassed if the stack canary can be leaked or brute-forced. The attacker might also be able to bypass canaries by altering the program flow in some other way that is not overwriting the return address (for example, by overwriting an important control variable of the function instead).

#### Address space layout randomization (ASLR)

ASLR randomizes memory layout and makes the addresses of the stack, the heap, and libraries unpredictable. This prevents attackers from easily predicting which memory address to jump to and makes code execution attacks harder.

#### Executable space protection

Another way to protect against overflow-based code execution is to mark areas of the memory as non-executable. This means that even if an attacker manages to plant malicious code, it would be treated as data, and not code, so the program would not execute it.

However, this method does not make the program completely immune to code execution either: attackers may still use techniques like [return-oriented programming](https://en.wikipedia.org/wiki/Return-oriented_programming) to accomplish the same goal.

### Secure coding practices to prevent buffer overflow

#### Input size checking

Input size checking should be implemented to ensure that user input can be contained within the allocated buffer space. In particular, user input that writes into arrays and format strings should be handled with care.

#### Use safe functions

Developers should trade functions which are not bounds-checked for functions that are. For example in C, printf(), sprintf(), strcat(), strcpy(), and gets() are unsafe, while snprintf(), strncpy(), strncat(), and fgets() are safe.