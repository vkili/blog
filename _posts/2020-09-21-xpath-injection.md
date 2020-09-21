---
title: "Hacking XML Data"
categories:
  - Hacking
---

Obtaining illegal data access using XPATH injections.

Code injection is a vulnerability with many faces: from SQL injection to OS command injection. These attacks happen because of a common programming mistake: letting user input pollute executable code.

Today, let's talk about a lesser-known type of code injection: injecting into XPATH queries.

## What is XPATH?

XPATH is a query language used for XML documents. Think SQL for XML.

XPATH provides the ability to navigate around the XML document tree, and select specific elements based on certain criteria.

For example, given an XML document:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Employees>
    <employee id="1">
      <name>Kacey</name>
      <title>Engineer</title>
    </employee>
    <employee id="2">
      <name>Aaron</name>
      <title>Engineer</title>
    </employee>
</Employees>
```

The XPATH expression below will select the ids of all employees:

```
/Employees/employee/@id
```

While this XPATH expression will select the names of all employees:

```
/Employees/employee/name/text()
```

As you can see, XPATH is very similar to SQL in terms of functionality, albeit with a slightly different syntax. The basic syntax of XPATH is kind of like navigating the XML document using a file path.

One major difference between XPATH and SQL is that XPATH is a standard language, and is not implementation-dependent. Whereas SQL has many different SQL dialects like MySQL, MSSQL, PostgreSQL, and SQLite. This difference is significant because it means that exploiting XPATH injection vulnerability is easier and potentially more scalable than exploiting SQL injection vulnerabilities because attackers won't have to customize their payloads according to the dialect.

### What is XPATH used for?

XPATH can be used to query and perform operations on data stored in XML documents. For example, XPATH can be used to retrieve salary information of employees stored in an XML document, and can also be used to perform numeric operations or comparisons on that data.

XML documents are used as databases for their portability, and their flexible and compatible structure.

From a security point of view, it is important to note that in real-life applications, user data is seldom stored in XML documents. But communicating sensitive data across systems and web services is often done using XML. So these places are more often vulnerable.

## Attacking XPATH queries

XPATH injection is an attack that injects into XPATH expressions in order to alter the outcome of the query. Similar to SQL injection, it can be used to bypass business logic, escalate user privilege, and leak sensitive data.

XPATH injection flaws occur when developers form dynamic XPATH queries using user input. Let's say we're working with an XML document like this: (Notice that Kacey is an admin while Aaron is not.)

```xml
<?xml version="1.0" encoding="utf-8"?>
<Employees>
    <employee id="1">
      <name>Kacey</name>
      <title>Engineer</title>
      <username>kacey1</username>
      <password>s3cret</password>
      <admin>1</admin>
    </employee>
    <employee id="2">
      <name>Aaron</name>
      <title>Engineer</title>
      <username>aaron2</username>
      <password>p4ssw0rd</password>
      <admin>0</admin>
    </employee>
</Employees>
```

A piece of code like this is vulnerable, as it concatenates user input into an XPATH expression to authenticate users.

```
Employees.SelectNodes("
//employee
[@username='" + USERINPUT.username + "'
and @password='" + USERINPUT.password + "']")
```

During authentication, all is well if the user does not attempt anything funky and simply provides their username and password:

```
Employees.SelectNodes("
//employee
[@username='aaron1'
and @password='p4ssw0rd']")
```

But if the user is malicious and attempts to alter the logic of the query by providing the username ' or 1=1 or ''=' to mess with the XPATH processor:

```
Employees.SelectNodes("
//employee
[@username='' or 1=1 or ''=''
and @password='any password']")
```

Since 1=1 is always true, the query would simply select the first employee in the document tree, in this case, Kacey. And because Kacey has admin privileges on the application, the attacker gains admin privileges as well.

### Implications of XPATH injection

XPATH injection has very serious implications, just like SQL injection. But there is one key difference between SQL and XPATH that potentially make XPATH injection even more dangerous.

SQL databases are often protected by user-based access controls: the user might be limited to certain tables, columns, and queries based on the rights of the database user that the application runs on. Whereas within a single XML document, there is no concept of access control, and XML databases have no concept of users or permissions. This means that a single XPATH injection flaw often leads to the compromise of the entire XML database.

### XPATH 2.0

The basics of XPATH injection applies to both XPATH 1.0 and XPATH 2.0. However, XPATH 2.0 expands the capabilities of XPATH 1.0, making XPATH injection vulnerabilities even more dangerous.

XPATH 2.0 is much more feature-rich compared to its older version. For security purposes, two features stand out in their exploitation potential.

First, XPATH 2.0 allows users to reference documents by URL. This means the target of exploitation is no longer limited to the current document, and attackers can try to retrieve an XML document whose location on the host is known and accessible to the current server.

XPATH 2.0 also has a function that converts a string to a sequence of unicode numbers that represents that string. This simplifies the extraction of string data during exploitation.

## Preventing XPATH injections

There are two ways that an application can protect against XPATH injection attacks: input sanitization and parameterized queries.

### Input sanitation and escaping

Applications can sanitize user input before inserting the input into an XPATH query to prevent XPATH injection attacks. For example, single and double quote characters should be disallowed or escaped to prevent user input from breaking out of the query string.

### Using parameterized queries

A better way to prevent XPATH injections is with parameterized queries: instead of building XPATH queries dynamically from user input, use parameters to insert user input into the query instead.

So instead of using:

```
Employees.SelectNodes("
//employee
[@username='" + USERINPUT.username + "'
and @password='" + USERINPUT.password + "']")
```

Use this instead:

```
Employees.SelectNodes("
//employee
[@username= $username
and @password= $password]")
```

This is a much safer way since input sanitation can sometimes be bypassed by using obscure characters, and its often difficult for developers to account for all the possible bypasses.

XPATH injection, like SQL injection, is a very devastating vulnerability. But just like SQL injection, there are clear cut ways to prevent them from happening. As a developer, be sure to used parameterized queries in your XPATH expressions! And for pentesters, look out for where XML data is queried and test for possible XPATH injection vulnerabilities.

Hope you enjoyed learning about XPATH injection attacks. Thanks for reading.
