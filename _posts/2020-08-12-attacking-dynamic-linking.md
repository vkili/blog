---
categories:
  - Binary Exploitation
---

And how modern binaries protect against attacks

There are two types of binaries: statically-linked binaries and dynamically-linked binaries.

Statically-linked binaries do not depend on external libraries. All the code and dependencies needed is contained within the binary itself. Whereas dynamically-linked binaries rely on shared-libraries for parts of their functionality. This allows processes to share code and eliminates the need to store shared library code in every executable.

## How it works

When a dynamically-linked ELF binary calls a function in a shared library (say, printf), the call actually points to the Procedure Linkage Table (PLT, located at the .plt section) within the binary. The PLT contains code stubs that direct the program to lookup function addresses stored in the Global Offset Table (GOT, located at the .got.plt section). The GOT will either contain a pointer back to the PLT to invoke the dynamic linker, or a pointer pointing to the shared library function (the actual address of printf).

The first time a function is called, the dynamic linker would be called to locate the actual address of the library function. That location will then be written into GOT. Then, in later calls to the library function, the program would be able to directly retrieve the location of the function from the GOT.

This process is called "lazy binding". Since binding a function (locating it in the library and writing its address to the GOT) is very computationally expensive, lazy binding allows programs to defer that cost to when the function is actually called. (Sometimes programs may include a shared function but never call it, depending on program control flow. In this case, the computational time used to bind that function is wasted.)

## Attacking the Dynamic Linking Process

You might already see what the problem is: the GOT is essentially a giant pointer array that points to functions in shared libraries. And because by default, the addresses in GOT is lazily bound during the program run, the GOT has to be writable.

And if the GOT is loaded at a fixed address (the binary is not compiled as [PIE](https://en.wikipedia.org/wiki/Position-independent_code)), it becomes possible for attackers to locate and overwrite addresses in the GOT. Once the attacker finds a bug that allows arbitrary write of a single address (typically through a format string vulnerability), the attacker can overwrite an address in the GOT and redirect the program flow to another function or a ROP gadget. For example, the attacker can make the program call system() instead of printf(). This is called a GOT overwrite attack.

## Relocation Read-Only (RELRO)

Relocation read-only is a mitigation technique used to prevent GOT overwrites. RELRO forces the linker to resolve all dynamically linked functions at the start of program execution and populate the GOT. It then marks the GOT as read-only, so that it cannot be modified afterwards.

If full RELRO is used, attempts to overwrite the GOT section will cause the program to crash, therefore preventing arbitrary code execution. However, this setting can greatly increase program startup time because all symbols will need to be resolved in advance.

## Other Potential Risks

Even if the attacker is not able to overwrite the GOT section, the GOT and the PLT remains an attractive target to redirect program flow. Especially when they link to important functions such as system() and execve().

When their addresses are not randomized, these sections can be a way for an attacker to bypass other binary exploitation mitigation techniques such as Address Space Layout Randomization. (More on these issues in later posts, so stay tuned!)