---
title: "Intro To Bash Scripting"
categories:
  - Bash Scripting
---

Using Bash Scripts To Automate Your Workflow.

If you work with computers, there's probably been a time where you've thought: "Wow, there's got to be a way I can automate this".

A computer script is a list of commands designed to be executed by a program. They are used to automate tasks such as data analysis, webpage generation, and system administration. And the ability to understand and create scripts is one of the most sought after skills in the IT world.

So today, let's talk about scripts. We will talk about bash scripts in particular, what they are, and why you should use them. Then, we will write and run a simple bash script of our own.

## What Is A Bash Script?

Bash stands for "Bourne Again Shell". It is a shell interpreter that takes in commands from the user and performs actions using operating system services.

For example, when you open up the terminal and type in the command "ls", you will see a listing of the contents of the current directory. What you see here is the shell interpreter at work.

The bash interpreter can also read commands from a file that the user has previously written and saved. This is called a "bash script": a file composed of a list of shell commands.

### What is it used for?

You might wonder: so why not just use the terminal directly? Instead of simply typing in commands into the terminal, why should I store it in a script?

Bash scripts, or any type of shell script, is useful for managing complexities and for automating recurrent tasks.

If your commands involve multiple input parameters, or when the input of one command depends on the output of another, things could get complex very quickly. And doing it all manually increases the chance of a programmer mistake.

On the other hand, you might have a list of commands that you want to execute many, many times. Scripts are useful here as it saves you the trouble of typing the same commands over and over again. Instead, you can just run the script and be done with it.

## Writing Your First Script

Now, let's get started with our first script. Open up your favorite text editor and follow along!

### The first line

The first line of your script should be the "shebang" line. It starts with a hash character (#) and a bang character (!) and it declares the interpreter for the script. This allows the plain text file to be executed like a binary.

This first line indicates that we are using bash as our interpreter:

```bash
#!/bin/bash
```

### Add some commands

For our first script, let's make our program says "Hello" to the user.

In Unix, the command "echo" prints out its arguments to standard output. So to make our program say "Hello!", we add a line to our script:

```bash
#!/bin/bash
echo "Hello!"
```

You can also let users provide input arguments in a bash script. In bash syntax, "$1" is the variable name for the first argument passed in, "$2" is the second argument, and so on...... And "$@" is the variable name for all arguments passed in.

Let's say we want our script to customize its welcome, so we allow our users to pass in their names as arguments. We will then print out a custom welcome for them:

```bash
#!/bin/bashecho "Hello!"
echo "Welcome to the script :) "
echo $@
```

At this point, our simple script is complete! Save it in your current directory with the file name "sayhi.sh". (The ".sh" extension is conventional for shell scripts.)

### File permissions

For security purposes, most files are not executable by default. You can make the shell script executable by running the command:

```bash
chmod +x sayhi.sh
```

The "chmod" command edits the permissions for a file, and "+x" indicates that we want to add the permission to execute for all users.

### Executing the script

Now we can run our script! You can do that by executing this command. (Note that we are passing in "Vickie" as the first argument, and "Jennifer" as the second argument!)

```bash
./sayhi.sh Vickie Jennifer
```

You should see the output of the script printed out:

```bash
Hello!
Welcome to the script :)
Vickie Jennifer
```

## Conclusion

And that's it for our first view into bash scripts! If you are a hacker, programmer, or system administrator, bash scripts would be extremely useful to help you optimize your workflow. Happy scripting.