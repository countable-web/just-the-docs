---
layout: default
title: Python
parent: Programming
nav_order: 11
---

# Python

**Purpose**

Capture our coding standards for Python projects, alongside other things that are useful to know when working on Python codebases at Countable.

**Scope**

Aggregation of all Python-specific standards, best practices, and known problems we may encounter.

## Python Code Standards

For Python code, please comply with PEP-8. Editor plugins can automate most of the rules for you. Common mistakes:

  - Use the `black` autoformatter.
  - [Functions should be snake\_case](https://www.python.org/dev/peps/pep-0008/#function-names)
  - [Classes should be UpperCamelCase](https://www.python.org/dev/peps/pep-0008/#class-names)
  - [Constants should be ALL\_CAPS](https://www.python.org/dev/peps/pep-0008/#id48)

### Imports

Order imports as follows

    # Builtins
    import os
    import sys
    import csv
    
    # 3rd Party
    import django
    
    # local
    import mymodule

### Exception Handling

Best practices for handling errors/ unexpected cases

    # bad - lost information about the error
    try:
       return 1/0
    except:
       print("I don't know what happened")
    
    # preferred - capture specific exception class
    try:
       1/0
    except ZeroDivisionError:
       print("now I know the exception")
    
    # avoid this when possible - catch-all and do a traceback with unexpected errors.
    try:
       code_that_could_fail()
    except Exception:
       import traceback
       print(traceback.format_exec())
    
    # better - use a custom exception for each response.
    try:
       code_that_could_fail()
    except CustomArithmeticError:
       print("Some kind of math error happened.")
    
    # bad - unnecessary lines in the try block
    try:
       a = 1
       b = a-a
       a/b
    exception ZeroDivisionError:
       print("One of these lines failed")
    
    # better - isolate which line you expect to fail 
    a = 1
    b = a-a
    try:
       a/b
    except ZeroDivisionError:
       print("Now we know which line failed")
    
    # bad - error does not report to the right places. team cannot easily fix. Params not sent.
    try:
       1/(a-b)
    except ZeroDivisionError:
       print("Divided by zero.")
    
    # better - code considers how error result is processed by humans, params sent.
    try:
       1/(a-b)
    except ZeroDivisionError:
       logging.error("Division by zero with params a={} b={b}, notifying the user of bad input.".format(a, b)) # Message for devs to observe if needed.
       raise HttpResponseBadRequest("Your parameters {}-{}=0, which is not allowed because it results in division by zero. Sorry!") # Notify the user what they did wrong.

**Key points for effective exception handling:**

   - Always catch specific exceptions rather than using a bare `except` clause.
   - Use logging for developer-oriented messages and user-friendly messages for end-users.
   - Keep the code in `try` blocks as minimal as possible to pinpoint the source of exceptions.
   - Use custom exceptions for better error categorization.
   - Include relevant parameters in error messages for easier debugging.

### Line Length

Lines of up to 119 chars are ok, instead of the default 79 chars. The reason for this is we have higher resolution screens these days and 79 chars is crazy restrictive.

## Names

  - attributes\_names - Attributes (and local variables) should use lowercase with underscores

## Common Gotchas

**Using lists and dicts in function parameters with default arguments**

When providing lists or dicts as default arguments in functions, you must not use an instantiated list/dict because _this causes that exact object to be shared each time you call the function_, instead of recreating a new copy of the list/dict on each function call. Instead, use `None` and instantiate the object inside the function.
```
def my_function(objects=[]):
    # bad
    ...


def my_function(objects=None):
    # correct
    if not objects:
      objects = []    
    ...
```

[Further reading](https://docs.python-guide.org/writing/gotchas/#mutable-default-arguments).


**Python Requests**

- By default, [Python Requests does not provide the `timeout` parameter](https://docs.python-requests.org/en/latest/user/advanced/#timeouts), which can cause timeouts in HTTP requests to bring down all Django workers in your app. It is recommended to always provide the `timeout` parameter when using Python Requests. A reasonable timeout time is 10 seconds, or longer (e.g. 30 seconds or more) if you're expecting a long-running request.

## Debugging Python

### Basics

  - You should become familiar with Python's debugger, `pdb` . Here are a few common cases with recipes.
  - If you want to inspect any object (variable) in python, use the `dir(obj)` function in the python CLI (\>\>\>)
  - If you want to know the source file of any module, do `import module` and then `module.__file__`

**There is a trace in Library code and you want to look around**

1.  Look at the stack trace, and identify the file and line you want to break at.
2.  Add `import pdb; pdb.set_trace()` on that line and save the file.
3.  Run your program, and when that line is reached the debugger will start interactively in your terminal.

**The program is really unstable and you want to debug whenever it crashes.**

1.  Replace the executable `python` with `pdb` temporarily, in your Dockerfile or docker-compose.yml
2.  For example, `python manage.py runserver` becomes `pdb manage.py runserver`
3.  Now, restart your web (django) container in the foreground. ie)
    `docker-compose stop web && docker-compose up web`
4.  When the program crashes, your terminal will halt in a PDB interface and you can print variable names, etc.
