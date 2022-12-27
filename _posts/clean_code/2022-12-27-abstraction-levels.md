---
layout: post
title: "Clean code - Mixing levels of abstraction"
mermaid: true
categories: [clean_code,good_practices]
tags: [python,clean_code,pythonic]
---

Mixing levels of abstraction is typically harder to read and maintain. In this tutorial we'll give some examples of functions which have mixed levels of abstraction and techniques we could use to clean up code like this. The main take away message from this tutorial is:

> Functions should do work thats one level abstraction below their name.
{: .prompt-tip}

This is not to be confused that a function can't have many other functions called inside its body. It's just the level of abstraction is consistent throughout. Low level code is fine as long as its written inside a well named function that gives the interpretation of that low level code.

Lets first take a look at a bad example.


---
## The problem

For this purpose, I'm going to give a basic example of a function that attempts to save a user or log the error message if the email is incorrect.

```python
def save_user(email: str) -> None:
    if "@" not in email:
        log.error("Invalid email")
    else:
        user = User(email)
        user.save()
```

Now I know that typically checking if an email is valid requires more rigorous checks but for the sake of simiplicity we assume that if an email doesn't have an `@` symbol its incorrect.

Notice that here we have some low level checks such as `if "@" not in email` and then we log the error. We also have to instantiate a `User` class and then call the save method. However, if we look at the name of the function, it should just take in a user email and have calls to functions one level lower that handles this, rather than doing the validation checks at this level.


---
## The solution

So we can start off by abstracting away the validation checks. Now our code becomes:

```python
def email_is_valid(email: str) -> bool:
    return "@" in email

def save_user(email: str) -> None:
    if not email_is_valid(email):
        log.error("Invalid email")
    else:
        user = User(email)
        user.save()
```

We kept the `email_is_valid` function "truthy", i.e. we are checking for positivity rather than negativity like `email_is_not_valid` as its typically easier to read. We're not quite done because we still have mixed level of abstractions. We are calling methods like `log.error()` and `user.save()`. The `save_user` class doesn't need to know how these implementations are done, just that it *is* done. So lets go ahead and abstract this away.


```python
def email_is_valid(email: str) -> bool:
    return "@" in email

def save_new_user(email) -> None:
    user = User(email)
    user.save()

def log_error(message) -> None:
    log.error(message)

def save_user(email: str) -> None:
    if not email_is_valid(email):
        log_error("Invalid email")
    else:
        save_new_user(email)
```

We wrapped the `log.error` in a separate function call `log_error`. This is handy because if we choose to change log handler in the future we won't have to make that change everywhere in our code, so we've locallised the problem to a single function. This is a good habbit to get into. As with the `save_new_user`, now we have delegated the responsibility of creating a new user and saving it to another function. This way `save_user` is unaware of the steps required (i.e. creating a new user object etc) but is just responsible for calling the `save_new_user` for this to happen.

One could argue that we are pretty much done with the `save_user` function. I would like to take it a step further and say that whilst the abstraction levels seem consistent, its not quite one level lower because we are still doing validation checks on the email. The `save_user` function shouldn't be responsible for this implementation of `if` checks.

```python
def email_is_valid(email: str) -> bool:
    return "@" in email

def save_new_user(email) -> None:
    user = User(email)
    user.save()

def log_error(message) -> None:
    log.error(message)

def validate_email(email: str) -> None:
    if not email_is_valid(email):
        log_error("Invalid email")
        raise ValueError("Invalid email")

def save_user(email: str) -> None:
    validate_email(email)
    save_new_user(email)
```

Now we've separated the concern of validating an email to the `validate_email` function which will raise a `ValueError` if its incorrect (at this point you would raise a custom error such as `EmailInvalidError` or something similar). But our `save_user` function is now much more readable and maintainable. The function has one level lower abstraction and delegates the appropriate calls to other functions which has that lower level implementation.

Thanks for reading!