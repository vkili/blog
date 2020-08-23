-
categories:
  - Bash Scripting
-

Writing functions to simplify your script.

If you work with computers, there's probably been a time where you've thought: "Wow, there's got to be a way I can automate this".

A computer script is a list of commands designed to be executed by a program. They are used to automate tasks such as data analysis, webpage generation, and system administration. And the ability to understand and create scripts is one of the most sought after skills in the IT world.

In the first three parts of this tutorial, we talked about the basic syntax of bash scripts, and how to use conditionals and loops in bash.

Today, let's dive deeper into bash scripts, and learn about how to write functions in Bash!

## Our script so far

Last time, we built an age calculator that calculates users' ages in days, months, weeks, and years.

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

Our script is named "calculate_age.sh". It takes an input of a MODE, a BIRTHDATE, and multiple NEW_DATES. MODE refers to the unit used to calculate age. It can be either days, weeks, months, or years.

If one or more NEW_DATE is provided, the program will calculate the user's age at every NEW_DATE. And if NEW_DATE is not provided, our program will calculate the user's age at the current date. This gives users the ability to calculate how old they were in the past, and how old they will be in the future.

```bash
./calculate_age.sh MODE BIRTHDATE (NEW_DATE) (NEW_DATE)...
```

For example:

```bash
$ ./calculate_age.sh months "2000-06-09" "2020-05-07" "2021-06-08"
On 2020-05-07, you are 242 months old.
On 2021-06-08, you are 255 months old.
```

## Functions in Bash

But did you know that you can also write functions in Bash? Functions help make our scripts more succinct and manageable because they allow us to reuse code. For example, in our script, the bolded parts seem to be quite repetitive.

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

We can reduce the repetition using functions! The syntax of a bash function looks like this.

```bash
FUNCTION_NAME()
{
  DO_SOMETHING
}
```

The bolded parts of our code do two things. First, it calculates the time difference between the later date and the user's birthdate. Then, it converts that time difference into the desired unit. You can put the repeated code into a function.

```bash
time_difference()
{
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  DIFF=$(expr $DIFF / $UNIT)
}
```

Then, you can call the function like any other shell command within the script.

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
time_difference()
{
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  DIFF=$(expr $DIFF / $UNIT)
}
if [ $3 ]
then
  for i in "${@:3}"
    do
      NEW_DATE=$(date -jf "%Y-%m-%d" $i +%s)
      time_difference 
      echo "On $i, you are $DIFF $MODE old."
  done
else
  NOW=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
  time_difference 
  echo "You are $DIFF $MODE old."
fi
```

Since we are using $NEW_DATE instead of $NOW inside the function, we need to quickly rename our variable in the bolded line. This will not affect any of the script's functionality.

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
time_difference()
{
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  DIFF=$(expr $DIFF / $UNIT)
}
if [ $3 ]
then
  for i in "${@:3}"
    do
      NEW_DATE=$(date -jf "%Y-%m-%d" $i +%s)
      time_difference 
      echo "On $i, you are $DIFF $MODE old."
  done
else
  NEW_DATE=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
  time_difference 
  echo "You are $DIFF $MODE old."
fi
```

You can see that we've reduced some repetition in our code. This simplification might not seem like much here, but reusing code using functions will save you a lot of headaches when you write more complex programs.

Let's look at another example! Say, you want to write a script that calculates your friends' ages. Our script, "friend_age.sh" will calculate our friends' age at each date.

```bash
$ ./friend_age.sh months "2000-06-09" "2020-05-07" "2021-06-08"
```

Theoretically, we can write our script like this:

```bash
#!/bin/bash
CHARLIE_BDAY= "1990-06-09"
SEAN_BDAY= "1992-06-09"
JEN_BDAY= "1998-06-09"
MODE=$1
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
esacecho "Calculating Charlie's age..."
BIRTHDATE=$CHARLIE_BDAY
time_difference()
{
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  DIFF=$(expr $DIFF / $UNIT)
}
if [ $3 ]
then
  for i in "${@:3}"
    do
      NEW_DATE=$(date -jf "%Y-%m-%d" $i +%s)
      time_difference 
      echo "On $i, you are $DIFF $MODE old."
  done
else
  NEW_DATE=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
  time_difference 
  echo "You are $DIFF $MODE old."
fi
echo "Calculating Sean's age..."
BIRTHDATE=$CHARLIE_BDAY
time_difference()
{
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  DIFF=$(expr $DIFF / $UNIT)
}
if [ $3 ]
then
  for i in "${@:3}"
    do
      NEW_DATE=$(date -jf "%Y-%m-%d" $i +%s)
      time_difference 
      echo "On $i, you are $DIFF $MODE old."
  done
else
  NEW_DATE=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
  time_difference 
  echo "You are $DIFF $MODE old."
fi
echo "Calculating Jen's age..."
BIRTHDATE=$SEAN_BDAY
time_difference()
{
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  DIFF=$(expr $DIFF / $UNIT)
}
if [ $3 ]
then
  for i in "${@:3}"
    do
      NEW_DATE=$(date -jf "%Y-%m-%d" $i +%s)
      time_difference 
      echo "On $i, you are $DIFF $MODE old."
  done
else
  NEW_DATE=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
  time_difference 
  echo "You are $DIFF $MODE old."
fi
```

But notice that there is so much repeated code! Thankfully, we can use a function to simplify our script. Our original script calculates the age of one person.

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
time_difference()
{
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  DIFF=$(expr $DIFF / $UNIT)
}
if [ $3 ]
then
  for i in "${@:3}"
    do
      NEW_DATE=$(date -jf "%Y-%m-%d" $i +%s)
      time_difference 
      echo "On $i, you are $DIFF $MODE old."
  done
else
  NEW_DATE=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
  time_difference 
  echo "You are $DIFF $MODE old."
fi
```

Since we are going through this procedure three times to calculate our friends' ages, we can wrap this script into a function and call it repeatedly.

```bash
#!/bin/bash
time_difference()
{
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  DIFF=$(expr $DIFF / $UNIT)
}
calculate()
{
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
        time_difference 
        echo "On $i, you are $DIFF $MODE old."
    done
  else
    NEW_DATE=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
    time_difference 
    echo "You are $DIFF $MODE old."
  fi
}
CHARLIE_BDAY="1990-06-09"
SEAN_BDAY="1992-06-09"
JEN_BDAY="1998-06-09"
echo "Calculating Charlie's age..."
calculate $1 $CHARLIE_BDAY ${@:3}
echo "Calculating Sean's age..."
calculate $1 $SEAN_BDAY ${@:3}
echo "Calculating Jen's age..."
calculate $1 $JEN_BDAY ${@:3}
```

Then, you can call any function in the script like any other shell command.

If you know how to program in other languages, you might be surprised by this script. Specifically, notice that all variables are global besides input parameters like $1, $2, and $3! Once a variable like $MODE is declared within a function, it becomes available throughout the script. And the variable remains available even when the function is exited. On the other hand, parameter values like $1, $2, and $3 refer to the values that the function is called with.

Let's try out our new script with a command.

```bash
$ ./friend_age.sh months "2000-06-09" "2020-05-07" "2021-06-08"
```

This above command will return:

```bash
Calculating Charlie's age...
On 2020-05-07, you are 364 months old.
On 2021-06-08, you are 377 months old.
Calculating Sean's age...
On 2020-05-07, you are 339 months old.
On 2021-06-08, you are 353 months old.
Calculating Jen's age...
On 2020-05-07, you are 266 months old.
On 2021-06-08, you are 280 months old.
```

## Writing a function library

As your codebase gets larger, you should consider wiring a function library to reuse code.

For example, we can store all the commonly used functions in a separate file called "common.lib".

```bash
#!/bin/bash
# this file is common.lib

time_difference()
{
  DIFF_IN_SECONDS=$(expr $NEW_DATE - $BIRTHDATE)
  DIFF=$(expr $DIFF_IN_SECONDS / 86400)
  DIFF=$(expr $DIFF / $UNIT)
}
calculate()
{
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
        time_difference 
        echo "On $i, you are $DIFF $MODE old."
    done
  else
    NEW_DATE=$(date -jf "%a %b %d %T %Z %Y" "$(date)" +%s)
    time_difference 
    echo "You are $DIFF $MODE old."
  fi
}
```

In another file, we can call the library file to use all of its functions and variables. We import a script via the "dot" command, followed by the path to the script.

```bash
#!/bin/bash
# this file is friend_age.sh
. ./common.lib
CHARLIE_BDAY="1990-06-09"
SEAN_BDAY="1992-06-09"
JEN_BDAY="1998-06-09"
echo "Calculating Charlie's age..."
calculate $1 $CHARLIE_BDAY ${@:3}
echo "Calculating Sean's age..."
calculate $1 $SEAN_BDAY ${@:3}
echo "Calculating Jen's age..."
calculate $1 $JEN_BDAY ${@:3}
```

Using a library can be super useful when you are building multiple tools that require the same functionalities. For example, you might build multiple networking tools that all require DNS resolution. In this case, you can simply write the functionality once and use it in all of your tools.