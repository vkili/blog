---
title: "Bash Variables And Conditionals"
categories:
  - Bash Scripting
---

Using variables and conditionals in Bash.

If you work with computers, there's probably been a time where you've thought: "Wow, there's got to be a way I can automate this".

A computer script is a list of commands designed to be executed by a program. They are used to automate tasks such as data analysis, webpage generation, and system administration. And the ability to understand and create scripts is one of the most sought after skills in the IT world.

Last time, we talked about bash scripts, what they are, and why you should use them. You even wrote a simple bash script of your own!

Today, let's dive deeper into bash scripts. You will learn how to use variables and conditionals with bash, and how you can use them to build scripts that do more.

## Setting up

For this tutorial, let's build a program that calculates the user's current age in days. The program takes the user's birthdate as input and outputs the user's age in days as of today.

As usual, the first line should be the "shebang" line. It declares bash as the interpreter for your script.

```bash
#!/bin/bash
```

## Variables

Next, you need to give users a way to input their birthdate. Since you need to operate on the user's birthdate later, we can store it in a convenient variable name. But what do variables look like in bash?

In bash syntax, "$1" is the variable name for the first argument passed in, "$2" is the second argument, and so on...... And "$@" is the variable name for all arguments passed in.

So we can reference "$1" to get the user's input and store it in a variable named "BIRTHDATE".

```bash
#!/bin/bash
BIRTHDATE=$1
```

Variables in bash can be assigned by using the below syntax. Note that there are no spaces around the "="!

```bash
VARIABLE_NAME=VARIABLE_VALUE
```

### Operating on variables

But to compute the time difference between two dates, we need to convert the user-supplied date string into numbers.

Assuming that the user's input string will be in the format of "2020--05--06", we can convert the user's birthdate to a Unix timestamp. Unix timestamps are a standardized way to represent a moment in time. It is the number of seconds that have passed since the "Unix epoch", 1970/1/1, 00:00:00 UTC.

```bash
#!/bin/bash
BIRTHDATE=$(date -jf "%Y-%m-%d" $1 +%s)
```

Let's break this down a bit. The `$()` in the second line tells Unix to execute the command surrounded by the parenthesis and replace the variable with the output of the command. This is called "command substitution" and is a feature that you will use very often.

`date` is a command that deals with displays time on Unix systems. You can use the `-jf` flag to convert one date format to another. `'%Y-%m-%d'` tells `date` that the input date string will be in the format of `year-month-day`. `$1` specifies that the input string is the first argument of our script. And finally, `+%s` tells `date` to output the date as the number of seconds that have passed since the "Unix epoch".

Next, we have to convert the current time into a Unix timestamp as well, so that we can calculate the difference between now and the user's birthdate. The output of the `$(date)` is the current time in this format: `Thu May 7 11:59:13 CDT 2020`. And this command will convert the current time into a Unix timestamp:

```bash
date -jf "%a %b %d %T %Z %Y" "$(date)" +%s
```

We can then place the output of this command into a variable.

```bash
#!/bin/bash
BIRTHDATE=$(date -jf "%Y-%m-%d" $1 +%s)
NOW=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
```

Next, let's calculate the difference between the two dates in seconds. To use the value of a variable, we reference it with `$VARIABLE_NAME`. We can calculate the difference between the two timestamps by using `expr`, a command that performs arithmetic operations.

```bash
#!/bin/bash
BIRTHDATE=$(date -jf "%Y-%m-%d" $1 +%s)
NOW=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
DIFF_IN_SECONDS=$(expr $NOW - $BIRTHDATE)
```

Finally, we convert the difference in seconds to a difference in days by dividing the time difference in seconds by 86400 (there are 86400 seconds in a day). And use `echo` to display the time difference.
```bash
#!/bin/bash
BIRTHDATE=$(date -jf "%Y-%m-%d" $1 +%s)
NOW=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
DIFF_IN_SECONDS=$(expr $NOW - $BIRTHDATE)
DIFF_IN_DAYS=$(expr $DIFF_IN_SECONDS / 86400)
echo "You are $DIFF_IN_DAYS days old."
```

Conditionals
============

Let's say that you also want your users to be able to calculate how old they will be in the future. We can give users the option of providing a second date. And if the second date is provided, the program will calculate the difference in days between the two dates instead of using the current time.

We can implement this with an `if` statement. In bash, the syntax of an `if` statement is as follows. Note that the ending of the conditional statement, `fi`, is "if" backward.

```bash
if [ condition 1]
then
  # do if condition 1 is satisfied
elif [condition 2]
  # do if condition 2 is satisfied, and condition 1 is not satisfied
else
  # do something else if neither condition is satisfied
fi
```

Let's set up a condition where if the second input argument exists, we calculate the date difference between the birthdate and the new date instead.

```bash
#!/bin/bash
BIRTHDATE=$(date -jf "%Y-%m-%d" $1 +%s)
if [ $2 ]
then
  NEW_DATE=$(date -jf "%Y-%m-%d" $2 +%s)
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
else
  NOW=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
  DIFF_IN_SECONDS=$(expr $NOW - $BIRTHDATE)
fi
  DIFF_IN_DAYS=$(expr $DIFF_IN_SECONDS / 86400)
echo "You are $DIFF_IN_DAYS days old."
```

### Case statements

Finally, let's add a few more features to your program! We can give users the ability to calculate their age in months, weeks, and years instead of just days. We will change the command syntax of our program to:

```bash
./calculate_age.sh MODE BIRTHDATE (NEW_DATE)
```

Where MODE refers to the unit to calculate age with. It can be one of days, weeks, months, or years. And the NEW_DATE is optional.

First, we need to change the input variable of BIRTHDATE and NEW_DATE to $2 and $3, since they are now the second and third input arguments.

```bash
#!/bin/bash
BIRTHDATE=$(date -jf "%Y-%m-%d" $2 +%s)
if [ $3 ]
then
  NEW_DATE=$(date -jf "%Y-%m-%d" $3 +%s)
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
else
  NOW=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
  DIFF_IN_SECONDS=$(expr $NOW - $BIRTHDATE)
fi
DIFF_IN_DAYS=$(expr $DIFF_IN_SECONDS / 86400)
echo "You are $DIFF_IN_DAYS days old."
```

Next, how do we perform different operations based on the MODE that the user has provided? We can parse the argument and write code based on `if-else` statements: if MODE is "days", else if MODE is "weeks", and so on. But this approach makes the code complicated. Is there an alternative?

In bash, there is a concept called `case` statements. It allows you to match several values against one variable without going through a long list of `if-else` statements. The syntax of `case` looks like this. And note that the ending of the statement, `esac`, is "case " backward.

```bash
case $VARIABLE_NAME in
  case1)
    do something
    ;;
  case2)
    do something
    ;;
  caseN)
    do something
    ;;
esac
```

So, we can construct our conditionals like this:

```bash
#!/bin/bash
BIRTHDATE=$(date -jf "%Y-%m-%d" $2 +%s)
if [ $3 ]
then
  NEW_DATE=$(date -jf "%Y-%m-%d" $3 +%s)
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
else
  NOW=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
  DIFF_IN_SECONDS=$(expr $NOW - $BIRTHDATE)
fi
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
case $1 in
  days)
    ;;
  weeks)
    DIFF=$(expr $DIFF / 7)
    ;;
  months)
    DIFF=$(expr $DIFF / 30)
    ;;
  years)
    DIFF=$(expr $DIFF / 365)
    ;;
esac
echo "You are $DIFF $1 old."
```

## Run your script

Save your file with the name `calculate_age.sh`, and you can start calculating your age.

```bash
$ ./calculate_age.sh days "2000-06-09" "2020-05-07"
You are 7272 days old.
$./calculate_age weeks "2000-06-09" "2020-05-07"
You are 1038 weeks old.
$ ./calculate_age months "2000-06-09" "2020-05-07"
You are 242 months old.
```