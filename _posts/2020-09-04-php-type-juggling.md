---
title: "PHP Type Juggling Vulnerabilities"
categories:
  - Insecure Deserialization
---

How PHP's type comparison features lead to vulnerabilities, and how to avoid them.

Much like Python and Javascript, PHP is a dynamically typed language. This means that variable types are checked while the program is executing. Dynamic typing allows developers to be more flexible when using PHP. But this kind of flexibility sometimes causes unexpected errors in the program flow and can even introduce critical vulnerabilities into the application.

Today, let's dive into PHP type juggling, and how they lead to authentication bypass vulnerabilities.

## How PHP compares values

PHP has a feature called "type juggling", or "type coercion". This means that during the comparison of variables of different types, PHP will first convert them to a common, comparable type.

For example, when the program is comparing the string "7" and the integer 7 in the scenario below:

```php
$example_int = 7;
$example_str = "7";

if ($example_int == $example_str) {
 echo("PHP can compare ints and strings.")
}
```

The code will run without errors and output "PHP can compare ints and strings." This behavior is very helpful when you want your program to be flexible in dealing with different types of user input.

However, it is also important to note that this behavior is also a major source of bugs and security vulnerabilities.

For example, when PHP needs to compare the string "7 puppies" to the integer 7, PHP will attempt to extract the integer from the string. So this comparison will evaluate to True.

```php
("7 puppies" == 7) -> True
```

But what if the string that is being compared does not contain an integer? The string will then be converted to a "0". So the following comparison will also evaluate to True:

```php
("Puppies" == 0) -> True
```

Loose type comparison behavior like these is pretty common in PHP and many built-in functions work in the same way. You can probably already see how this can be very problematic, but how exactly can hackers exploit this behavior?

## How vulnerability arises

The most common way that this particularity in PHP is exploited is by using it to bypass authentication.

Let's say the PHP code that handles authentication looks like this:

```php
if ($_POST["password"] == "Admin_Password") {
	login_as_admin();
}
```

Then, simply submitting an integer input of 0 would successfully log you in as admin, since this will evaluate to True:

```php
(0 == "Admin_Password") -> True
```

### Conditions of exploitation

However, this vulnerability is not always exploitable and often needs to be combined with a deserialization flaw. The reason for this is that POST, GET parameters and cookie values are, for the most part, passed as strings or arrays into the program.

If the POST parameter from the example above is passed into the program as a string, PHP would be comparing two strings, and no type conversion would be needed. And "0" and "Admin_Password" are, obviously, different strings.

```php
("0" == "Admin_Password") -> False
```

However, type juggling issues can be exploited if the application takes accepts the input via functions like json_decode() or unserialize(). This way, it would be possible for the end-user to specify the type of input passed in.

```php
{"password": "0"}
{"password": 0}
```

Consider the above JSON blobs. The first one would cause the password parameter to be treated as a string whereas the second one would cause the input to be interpreted as an integer by PHP. This gives an attacker fine-grained control of the input data type and therefore the ability to exploit type juggling issues.

## Avoiding type juggling issues in PHP code

As a developer, there are several steps that you can take to prevent these vulnerabilities from happening.

### Use strict comparison operators

When comparing values, always try to use the type-safe comparison operator "===" instead of the loose comparison operator "==". This will ensure that PHP does not type juggle and the operation will only return True if the types of the two variables also match. This means that (7 === "7") will return False.

### Specify the "strict" option for functions that compare

Always consult the PHP manual on individual functions to check if they use loose comparison or type-safe comparison. See if there is an option to use strict comparison and specify that option in your code.

For example, PHP's [in_array()](https://www.php.net/manual/en/function.in-array.php) uses loose comparison by default. But you can make it switch to type-safe comparison by specifying the strict option.

If the function only provides loose comparison, avoid using that function and search for alternatives instead.

### Avoid typecasting before comparison

Avoid typecasting right before comparing values, as this will essentially deliver the same results as type juggling. For example, before typecasting, the following three variables are all seen as distinct by the type-safe operator.

```php
$example_int = 7;
$example_str = "7_string";
$example_str_2 = "7";

if ($example_int === $example_str) {
  # This condition statement will return False
  echo("This will not print.");
}

if ($example_int === $example_str_2) {
  # This condition statement will return False
  echo("This will not print.");
}
```

Whereas after typecasting, PHP will only preserve the number extracted from a string, and "7_string" will become the integer 7.

```php
$example_int = 7;
$example_str = "7_string";

if ($example_int === (int)$example_str) {
  # This condition statement will return True
  echo("This will print.");
}
```

## Conclusion

PHP is a great language that is flexible, convenient and easy to get started with. But this flexibility came at a cost. There are many features of PHP that can lead to vulnerabilities if the developer is not careful.

Type juggling is one of those features that have the potential to create critical vulnerabilities. Be extra careful when comparing values and always understand how your program is doing so.