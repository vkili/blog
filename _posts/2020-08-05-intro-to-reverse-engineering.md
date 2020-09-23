---
categories:
  - Binary Exploitation
---

Analyzing and Hacking Binaries with Ghidra.

Reverse engineering is a process that hackers use to figure out a program’s components and functionalities in order to find vulnerabilities in the program. You recover the original software design by analyzing the code or binary of the program, in order to hack it more effectively.

Today, let’s take a look at how to reverse engineer a single program using a piece of open-source software called Ghidra.

Note: This post is mostly dedicated to reverse engineering Linux C binaries. Please review your C programming skills before we move on! Also, it would be useful to download a practice program to analyze while following along with this post. A list of practice binaries can be found [here](https://crackmes.one/).

## Useful Binary Analysis Utilities
First, before we jump into using Ghidra, here are a few command-line utilities that you can use to gain information about a binary.

The “strings” command finds the printable strings in an object, binary or file.

```bash
strings <path to binary file>
```

The “file” command will reveal the file type of the file you are analyzing.

```bash
file <path to binary file>
```

## Getting Started With Ghidra
Ghidra is a software reverse engineering framework created by the National Security Agency (NSA) of the USA. It includes a variety of tools that helps users analyze compiled code on a variety of platforms including Windows, macOS, and Linux. Its capabilities include disassembly, assembly, decompilation, and many others.

Today, we are going to go through the features that would be most useful for a beginner in reverse engineering.

### Installing Ghidra
First, to install Ghidra, you’d have to download the files from Ghidra’s homepage [here](https://ghidra-sre.org/).
Ghidra also requires a supported version of a Java Runtime and Development Kit on the PATH to run. So you also need to install it. On Linux systems, you can simply run:

```bash
apt install openjdk-11-jdk
```

Then, you can either enter the path of the JDK directory when prompted during the first run of Ghidra, or you can add it directly to your PATH variable by running:

```bash
export PATH=<path of extracted JDK dir>/bin:$PATH
```

Finally, you should be able to run Ghidra’s executable in the unzipped Ghidra directory by running:

```bash
./ghidraRun
```

### First look at a binary
We can now start analyzing programs using Ghidra! To open a binary in Ghidra, you first create a new project by going to File > New project. Then go to File > Import file to import the binary file that you want to analyze.

![](https://vickieli.dev/blog/assets/images/binary-01.png)

After importing the file, you will see a window like this one:

![](https://vickieli.dev/blog/assets/images/binary-02.png)

You will also be able to see some basic info about the program you are analyzing:

![](https://vickieli.dev/blog/assets/images/binary-03.png)

### A quick tour around Ghidra
Double click on the file you want to analyze. This will open up a new window like this. This is the main window that we will be working with!

![](https://vickieli.dev/blog/assets/images/binary-04.png)

Let’s go through the different views in this window!

You can find a list of symbols in the Symbol Tree view on the left. Symbols are references to some type of data like an import, a global variable, or a function.

The Listing view in the middle shows typical assembly code fields like addresses, bytes, operands, and comments, etc.

Whereas the decompiler on the right converts assembly back to C code. To do that, you can simply double click on the function that you want to analyze in the symbol tree view.

![](https://vickieli.dev/blog/assets/images/binary-05.png)

### Finding a function
So how do we begin navigating around the binary and looking for individual functions? After all, we need to find interesting locations in the program to start our analysis!
For example, how do we find the “main” function of a program? First, you can try to search for the function in the symbol tree view:

![](https://vickieli.dev/blog/assets/images/binary-06.png)

But sometimes, the binary you are analyzing will not have symbols. This is called a “stripped binary”. What can you do to find the “main” function then?

Ghidra makes it easy for us as well. You can simply look for the arguments that the “\__libc_start_main” function is calling. One of them should be the address of the main function. 

This is because even when a binary is stripped and does not have symbols, it still uses dynamic libraries like “libc”. And so it needs to include the symbols that these libraries are using. Therefore, when libc tries to call the main function, Ghidra will be able to recognize its address. 

### Editing the program during analysis 
You can also edit the program during analysis in Ghidra. For example, you can edit a function by right-clicking on the function name in either the symbol tree, the listing window or the decompiler window, then going to the “Edit Function” option.

![](https://vickieli.dev/blog/assets/images/binary-07.png)

You can also retype and rename variables by right-clicking on the variable names and then going to the “Retype Variable” or “Rename Variable” option.

## Conclusion
Reverse engineering is a powerful method to analyze programs and to discover vulnerabilities. If you want to get into reverse engineering, Ghidra is a great tool to start with!

There are many more features in Ghidra that you can discover: including the ability to auto-analyze binary files of different platforms, and a variety of features designed to streamline the binary analysis process. In this post, we went through how to get started with Ghidra and the basics of analyzing a binary. Good luck with your journey using Ghidra!
