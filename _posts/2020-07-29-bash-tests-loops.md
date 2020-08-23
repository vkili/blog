---
title: "Bash Tests And Loops"
categories:
  - Bash Scripting
---

Using tests and loops in Bash.

If you work with computers, there's probably been a time where you've thought: "Wow, there's got to be a way I can automate this".

A computer script is a list of commands designed to be executed by a program. They are used to automate tasks such as data analysis, webpage generation, and system administration. And the ability to understand and create scripts is one of the most sought after skills in the IT world.

In parts one and two of this tutorial, we talked about the basic syntax of bash scripts, and how to use conditionals in bash.

Today, let's dive deeper into bash scripts. You will learn how to use loops, and how you can use them to build scripts that do more.

## Our script so far

Last time, we build an age calculator that gives users the ability to calculate their age in days, months, weeks, and years.

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

We saved the file with the name "calculate_age.sh" and users can calculate their age with the command below.

```bash
./calculate_age.sh MODE BIRTHDATE (NEW_DATE)
```

Where MODE refers to the unit used to calculate age with. It can be either days, weeks, months, or years. And NEW_DATE is optional. If NEW_DATE is provided, the program will calculate the difference in days between the two dates instead of using the current time. This gives users the ability to calculate how old they were in the past, and how old they will be in the future.

```bash
$ ./calculate_age.sh days "2000-06-09" "2020-05-07"
You are 7272 days old.
$./calculate_age.sh weeks "2000-06-09" "2020-05-07"
You are 1038 weeks old.
$ ./calculate_age.sh months "2000-06-09" "2020-05-07"
You are 242 months old.
```

## While Loops

What if we want users to be able to calculate ages at multiple points in time? Instead of just one date, users should be able to enter multiple dates in time and calculate their ages at those dates in one go.

```bash
./calculate_age.sh MODE BIRTHDATE (NEW_DATE) (NEW_DATE) ...
```

For example:

```bash
$ ./calculate_age.sh months "2000-06-09" "2020-05-07" "2020-06-08"
On 2020-05-07, you are 7272 days old.
On 2020-06-08, you are 7304 days old.
```

How do we implement this? We could use loops, for sure!

There are two types of loops in bash: the `while` loop and the `for` loop. Here's the syntax of a while loop. As long as the CONDITION is true, the while loop will execute the code between "do" and "done" repeatedly.

```bash
while CONDITION
do
  DO SOMETHING
done
```

Let's implement our new functionality using a while loop!

```bash
shift
shift
while test $# -gt 0
do
  NEW_DATE=$(date -jf "%Y-%m-%d" $1 +%s)
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  echo "On $1, you are $DIFF days old."
  shift
done
```

This block of code calculates and prints the age of the user in days on every NEW_DATE.

Let's break it down a bit! In the first two lines, the "shift" command tells bash to discard the first two command-line arguments passed in.

```bash
$ ./calculate_age.sh days "2000-06-09" "2020-05-07" "2020-05-08"
```

In this command, for example, `days` is `$1`, "2000-06-09" is `$2`, "2020-05-07" is `$3`, and "2020-05-08" is `$4`. After a "shift" command, "2000-06-09" becomes `$1`, "2020-05-07" becomes `$2`, and "2020-05-08" becomes `$3` .

In short, after every "shift" command, the first remaining command-line argument is discarded. We can use therefore use shift to iterate through the command-line arguments. The first two lines remove the first two arguments: the MODE and the BIRTHDATE. What remains are the NEW_DATES.

In the next line, `test $# -gt 0` is the condition that controls whether the while loop keeps running or not. The test command evaluates a conditional and outputs either true or false. `$#` refers to the number of arguments remaining after the shift commands. `-gt` is a comparison flag that stands for "greater than". So this line says tells the loop to keep going as long as the number of remaining arguments is greater than zero. You can find the list of comparison flags in the manual of the test command by running:

```bash
man test
```

Finally, we calculate the user's age on the next NEW_DATE and print out the results.

## For loops

What if we want to implement the functionality using a "for" loop? Here's the syntax of a for loop in Bash. For every item in LIST_OF_VALUES, Bash will execute the code between "do" and "done" once.

```bash
for i in LIST_OF_VALUES
do
  DO SOMETHING
done
```

Now, let's implement our functionality using a "for" loop!

```bash
for i in "${@:3}"
do
  NEW_DATE=$(date -jf "%Y-%m-%d" $i +%s)
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  echo "On $1, you are $DIFF days old."
done
```

In the first line of our loop, `"${@:3}"` is an array that contains every command-line argument starting from the third one. `$@` represents the array containing all input arguments, while `"${@:3}"` slices the array so that it contains only the NEW_DATES.

Array slicing is a way of extracting a subset of items from an array. In bash, you can slice arrays using the syntax:

```bash
"${INPUT_ARRAY:START_INDEX:END_INDEX}"
```

Our loop will use every command-line argument starting from the third one and use it to calculate an age.

## Putting it all together

Finally, let's combine our loop with the rest of our script.

```bash
#!/bin/bash
MODE=$1
BIRTHDATE=$(date -jf "%Y-%m-%d" $2 +%s)
case $MODE in
  days)
    UNIT=1
    ;;
  weeks)
    UNIT=7
    ;;
  months)
    UNIT=30
    ;;
  years)
    UNIT=365
    ;;
esac
if [ $3 ]
then
  for i in "${@:3}"
    do
      NEW_DATE=$(date -jf "%Y-%m-%d" $i +%s)
      DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
      DIFF=$(expr $DIFF_IN_SECONDS / 86400)
      DIFF=$(expr $DIFF / $UNIT)
      echo "On $i, you are $DIFF $MODE old."
  done
else
  NOW=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
  DIFF_IN_SECONDS=$(expr $NOW - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  DIFF=$(expr $DIFF / $UNIT)
  echo "You are $DIFF $MODE old."
fi
```

With our new script, users can now calculate their ages at multiple points in time.

```bash
$ ./calculate_age.sh months "2000-06-09" "2020-05-07" "2021-06-08"
On 2020-05-07, you are 242 months old.
On 2021-06-08, you are 255 months old.
```