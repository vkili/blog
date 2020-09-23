---
title: "Diving into unserialize()"
categories:
  - Insecure Deserialization
---

How PHP's unserialize() works, and why it leads to vulnerabilities.

## PHP's unserialize() function

In a nutshell, PHP's unserialize() function takes a string (representing a serialized object) and converts it back to a PHP object.

Basically, when you need to store a PHP object or transfer it over the network, you first use serialize() to pack it up.

```
serialize(): PHP object -> plain old string that represents the obj
```

Then when you need to use that data again, you use unserialize() to unpack and get the data that you want. Neato, am I right?

```
unserialize(): string containing object data -> original object
```

### The details

According to [PHP docs](https://www.php.net/manual/en/function.unserialize.php), unserialize() "creates a PHP value from a stored representation", and "takes a single serialized variable and converts it back into a PHP value".

It takes two parameters: str and options. str is the parameter containing the serialized string waiting to be deserialized. options is the array containing the options that control certain function behaviors. In unserialize() particularly, the only valid user-defined option is allowed_classes. allowed_classes specify the class names that should be accepted.

We'll dive into allowed_classes further, but essentially, when unserialize() encounters an object of a class that isn't to be accepted, then the object will be instantiated as \_\_PHP_Incomplete_Class instead.

### How it works

![](https://vickieli.dev/blog/assets/images/serialize-07.png)

#### Step 0: What are PHP magic methods?

PHP magic methods are function names in PHP that have "magical" properties. Learn more about them [here](https://www.php.net/manual/en/language.oop5.magic.php).

The magic methods that are relevant for us now are \_\_wakeup() and \_\_destruct(). If the class of the serialized object implements any method named \_\_wakeup() and \_\_destruct(), these methods will be executed automatically when unserialize() is called on an object.

#### Step 0.1: Unserialize prerequisites

When you serialize an object in PHP, serialize() will save all properties in the object. But it will not store the methods of the class of the object, just the name of the class.

Thus, in order to unserialize() an object, the class of the object will have to be defined in advance (or be autoloaded). That is, the definition of the class needs to be present in the file that you unserialize() the object in.

If the class is not already defined in the file, the object will be instantiated as \_\_PHP_Incomplete_Class, which has no methods and the object will essentially be useless.

#### Step 1: Object instantiation

Instantiation is when the program creates an instance of a class in memory. And that is what unserialize() does. It takes the serialized string, which specifies the class of the object to be created and the properties of that object. With that data, unserialize() creates a copy of the originally serialized object.

It will then search for a function named \_\_wakeup(), and execute code in that function if it is defined for the class. \_\_wakeup() is called to reconstruct any resources that the object may have. It is often used to reestablish any database connections that may have been lost during serialization and perform other reinitialization tasks.

#### Step 2: Program uses the object

The program can then operate on the object and use it to perform other actions.

#### Step 3: Object destruction

Finally, when no reference to the deserialized object instance exists, \_\_destruct() is called.

## How vulnerabilities happen in unserialize()

When an attacker controls a serialized object that is passed into unserialize(), she can control the properties of the created object. This will then allow her the opportunity to hijack the flow of the application, by controlling the values passed into automatically executed methods like \_\_wakeup().

This is called a PHP object injection. PHP object injection can lead to code execution, SQL injection, path traversal or DoS, depending on the context in which it happened.

For example, consider this vulnerable code snippet: (taken from <https://www.owasp.org/index.php/PHP_Object_Injection>)

```php
class Example2
{
   private $hook; 
   function __construct(){
      // some PHP code...
   } 
   function __wakeup(){
      if (isset($this->hook)) eval($this->hook);
   }
}
// some PHP code...
$user_data = unserialize($_COOKIE['data']);
// some PHP code...
```

An attacker can achieve RCE using this deserialization flaw because a user-provided object is passed into unserialize, and the class Example2 has a magic function that runs eval() on user-provided input.

To exploit this RCE, the attacker simply has to set her data cookie to a serialized Example2 object with the hook property set to whatever PHP code that she wants executed. She can generate the serialized object using the following code snippet:

```php
class Example2
{
   private $hook = "phpinfo();";
}
print urlencode(serialize(new Example2));
```

In this case, passing the above-generated string into the data cookie will cause phpinfo() to be executed. Once the attacker passes the serialized object into the program, the following is what will happen in detail:

1.  The attacker passes a serialized Example2 object into the program as the data cookie.
2.  The program calls unserialize() on the data cookie.
3.  Because the data cookie is a serialized Example2 object, unserialize() instantiates a new Example2 object.
4.  unserialize() sees that the Example2 class has \_\_wakeup() implemented, so \_\_wakeup() is called.
5.  \_\_wakeup() looks for the $hook property of the object, and if it is not NULL, it runs eval($hook).
6.  $hook is not NULL, and is set to "phpinfo();", so eval("phpinfo();") is run.
7.  RCE is achieved.

### Prerequisites for the exploit

There are a few conditions that have to be met for this to be exploitable. Let's look at this chart again, as it points to important exploit prerequisites:

![](https://vickieli.dev/blog/assets/images/serialize-07.png)

In order for an attacker to exploit an insecure unserialize() function, two conditions must be met:

-   The class of the deserialized object needs to be defined (or autoloaded) and needs to be one of the allowed classes.
-   The class of the object must implement some magic method that allows an attacker to inject code into.

And there you have it! That's how unserialize() leads to dangerous vulnerabilities.

## How to protect against unserialize() vulnerabilities

In order to prevent PHP object injections from happening, it is recommended to never pass untrusted user input into unserialize(). Consider using JSON to pass serializes data to and from the user. And if you do need to pass untrusted, serialized data into unserialize(), be sure to implement rigorous data validation in order to minimize the risk of a critical vulnerability.

-   Mitigation advice is taken from *php.net*. Read more about the unserialize() function [here](https://www.php.net/manual/en/function.unserialize.php).

* * * * *

As always, thanks for reading! Next time, we'll dive even deeper into the unserialize() function, and analyze some real-world vulnerabilities!