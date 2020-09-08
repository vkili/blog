---
title: "PHP Magic Methods"
categories:
  - Insecure Deserialization
---

Magic methods that can be used to kick start your RCE chain.


Previously, we talked about how PHP's unserialize leads to vulnerabilities, and how an attacker can utilize it to achieve RCE. This time, let's dive deeper into some of the most useful magic methods that you can use to achieve that coveted RCE.

## The unserialize() vulnerability, summarized

When an attacker controls a serialized object that is passed into unserialize(), she can control the properties of the created object. This will then allow her the opportunity to hijack the flow of the application, by controlling the values passed into magic methods like \_\_wakeup().

The attacker can then execute the code contained in the magic methods using the parameters specified by her, or use the magic methods as a means of starting a POP chain.

POP stands for Property Oriented Programming, and the name comes from the fact that the attacker can control all of the properties of the deserialized object. Similar to ROP attacks (Return Oriented Programming), POP chains work by chaining code "gadgets" together to achieve the attacker's ultimate goal. These "gadgets" are code snippets borrowed from the codebase that the attacker uses to further her goal.

## The magic methods

There are four magic methods that are particularly useful when trying to exploit an unserialize() vulnerability: \_\_wakeup(), \_\_destruct(), \_\_toString() and \_\_call(). Today, we're gonna discuss

-   What they are
-   What they do
-   And why they are useful for constructing exploits.

## \_\_wakeup()

\_\_wakeup() is a magic method that is invoked on unserialize(). It is normally used to reestablish any database connections that may have been lost during serialization and perform other reinitialization tasks.

It is often useful during an unserialize() exploit because if it is defined for the class, it is automatically called upon object deserialization. Thus, it provides a convenient entry point to the database or to other functions in the code for POP chain purposes.

## \_\_destruct()

When no reference to the deserialized object instance exists, \_\_destruct() is called. It is invoked on garbage collection and is normally used to clean up references and finish other unfinished businesses associated with the object.

As it is used to clean up resources and shut down functionalities, it is very often found that \_\_destruct() contains useful code in terms of exploitation. For example, if a \_\_destruct() method contains code that deletes and cleans up files associated with the object, this might give the attacker an opportunity to mess with the integrity of the filesystem.

## \_\_toString()

Unlike \_\_wakeup() and \_\_destruct(), the \_\_toString() method is only invoked when the object is treated as a string. (Although if a \_\_toString() method is defined for the class, it is likely that it would get used somewhere.)

The \_\_toString() method allows a class to decide how it will react when it is treated as a string. For example, what will print if the object were to be passed into an echo() or print() function?

The exploitability of this magic method varies wildly, depending on how it is implemented. For example, here is a \_\_toString() function that could be used to start a POP chain: (This example is taken from [owasp.org](https://www.owasp.org/index.php/PHP_Object_Injection).)

```php
class Example3
{
  protected $obj; 
  function __construct()
  {
    // some PHP code...
  } 
  function __toString()
  {
    if (isset($this->obj)) return $this->obj->getValue();
  }
}
// some PHP code...
$user_data = unserialize($_POST['data']);
// some PHP code...
```

In this case, when an Example3 instance is treated as a string, it will return the result of the getValue() method of its $obj property. And since the $obj property is completely under the attacker's control, this will allow the attacker a lot of flexibility and will likely result in a critical vulnerability. \_\_toString() has also often been found to enable access to sensitive files.

## \_\_call()

The \_\_call() method is invoked when an undefined method is called. For example, calling $object->undefined($args) will turn into $object->\_\_call('undefined', $args).

Again, the exploitability of this magic method varies wildly, depending on how it was implemented. But it is often found to have a lot of potential when it comes to starting a POP chain after an insecure deserialization entry point.

## Other magic methods

Although these four magic methods are typically the most useful, there are, of course, many other methods that you can use to exploit an unserialize() vulnerability.

If the methods mentioned above are not exploitable, it might be worth it to check out the class' implementation of the other magic methods and if you can start an exploit chain from there. Read more about PHP's magic methods here: <https://www.php.net/manual/en/language.oop5.magic.php>.