---
layout: post
title: "Descriptors in python"
mermaid: true
categories: [design_patterns,descriptors, OOP]
tags: [python,design_patterns,composition,OOP]
---

Descriptors are useful abstractions (when used well) to help keep your code DRY. A descriptor is essentially a class that implements the descriptor protocol, that is, a class that has the following methods:

* `__get__()`
* `__set__()`
* `__delete__()` (Optional)
* `__set_name__()` (Optional)

When a user has only implemented the `__get__()` method for its descriptor, we define this as a `non-data descriptor`. When a user has extending this to the `__set__()` method (and optionally `__delete__()`), we say this is a `data descriptor`.

> *Due to the amount of boilerplate code required to implement a descriptor (typically), you should first seek to see if you can solve the problem in a simpler way (such as using properties) and then if further abstraction is needed, look into using descriptors.*
{: .prompt-tip}

# Attribute lookup

Before we dive into descriptors, lets look at an example of what normally occurs if we wanted to access the attributes of a class variable.

```python
>>> class Generic:
...     value = 5
...
>>> class Client:
...     generic = Generic()
...
>>> client = Client()
>>> client.generic.value
5
```

Here we assigned an instance of `Generic` to the `Client` class attribute named `generic`. In order for us to access `value`, we use the dot notation for attribute lookup, first on `generic` and then `value`. We can see by using vars the attributes available to use in `Client`

```python
>>> vars(Client)
mappingproxy({'__module__': '__main__', 'generic': <__main__.Generic object at 0x1054b4d50>, '__dict__': <attribute '__dict__' of 'Client' objects>, '__weakref__': <attribute '__weakref__' of 'Client' objects>, '__doc__': None})
```

The key `generic` has a mapping to an instance of `Generic`.

> For the rest of this tutorial, I will be refering to the client class as the user of the descriptor.
{: .prompt-info }

# Non-data descriptors

In order to setup a descriptor we need two classes, a user of the descriptor (usually denoted as the `client`) as well as the descriptor class. Examples are always a great way to see something in action, so lets write up our first descriptor that has static behaviour of always returning the value 42 when envoked.

```python
>>> class NonDataDescriptor: #1
...     def __get__(self, instance, owner): #2
...             return 42 #3
...
>>> class Client: #4
...     descriptor = NonDataDescriptor() #5
...
>>> Client().descriptor #6
42
```

To explain a bit whats happening here:

1. Defined a class `NonDataDescriptor` to be our descriptor 
2. Defined the `__get__` method to tell python this class is a descriptor. When we now do dot notation attribute lookup on the descriptor through the client class, this method is invoked.
3. Static value of 42 gets returned whenever the descriptor is invoked
4. Client class is now the requester of the descriptor.
5. Important to observe that a descriptor is assign as a class attribute NOT and instance attribute of the client that is, not inside the `__init__`.
6. As noted in `2.`, the descriptors __get__ method override attribute lookup here on the class attribute rather than returning an instance of the `NonDataDescriptor`. 

To the observer, one might be wondering what `instance` and `owner` represent in the signature of `__get__`. The `instance` parameter here represents the instance of the client, while the owner 
represents the type of the client instance, i.e. the class itself.

## Non-data descriptor real world example

Let's suppose we have a requirement to have a client class which manages employee data at our company (called `company`). Super simple, it takes in first and last name. We need a way of also creating that employee's email address dynamically. Sounds like we could use a descriptor that implements this logic in the `__get__` method.

```python
class EmailDescriptor: #1

    def __get__(self, instance, owner): #2
        if instance is None:
            return self
        email = f"{instance.first_name.lower()}.{instance.last_name.lower()}@company.com"
        return email 

class Employee: #3

    email = EmailDescriptor() #4

    def __init__(self, first_name, last_name) -> None:
        self.first_name = first_name
        self.last_name = last_name

if __name__ == "__main__":
    employee = Employee("John", "Doe") #5
    print(employee.email) #6
```

Here we have:

1. Our descriptor protocol for generating email addresses
2. The protocols getter uses the clients instance attributes `first_name` and `last_name` to generate an email address with the domain `company.com`
3. Our client class which uses the descriptor 
4. Associating the descriptor instance to the class attribute `email`
5. Creating a new employee object for John Doe
6. The attribute lookup here now invokes the EmailDescriptor.__get__ i.e. returns the email address `john.doe@company.com`.

> *The conditional `if instance is None`, is there to return the descriptor in the event `Employee.email` is called from the class. This safeguards from an error being raised because the `__get__` tries to access `first_name`, `last_name` which are only set on the instance.*
{: .prompt-tip}


On first observation, this seems to do the trick but looking a bit closer we've kinda closely coupled our client class (Employee) with our descriptor class (EmailDescriptor). What if we had another Employee class that needed to generate email addresses for `companyZ`.

Lets refactor and make our `EmailDescriptor` more abstract.

```python
class EmailDescriptor: #1

    def __init__(self, company: str) -> None:
        self.company = company #2

    def __get__(self, instance, owner):
        if instance is None:
            return self
        email = f"{instance.first_name.lower()}.{instance.last_name.lower()}@{self.company}.com" #3
        return email 

class EmployeeA: #4
    """Employee in Company A."""
    email = EmailDescriptor("companyA")

    def __init__(self, first_name, last_name) -> None:
        self.first_name = first_name
        self.last_name = last_name

class EmployeeB: #5
    """Employee in Company B."""
    email = EmailDescriptor("companyB")

    def __init__(self, first_name, last_name) -> None:
        self.first_name = first_name
        self.last_name = last_name

if __name__ == "__main__":
    employeeA = EmployeeA("John", "Doe")
    print(employeeA.email)
    employeeB = EmployeeB("Jane", "Doe")
    print(employeeB.email)
```

1. Descriptor constructor can accept a company domain name so that it can be reused against multiple `Employee` objects from different companies. This is far more abstract than our prevously coupled example.
2. Setting in the constructor the company name as an instance attribute. Please note, that whatever you put inside the `__init__` here will be invoked when the client class is defined. This will typically run when they are instantiated inside the client classes definition.
3. The returned value from the descriptor now includes the parametrised company name
4. Employee class for company A
5. Employee class for company B

## Data descriptors

In order to define a data descriptor, we also need to implement the `__set__` method in our descriptor class. Here we create a descriptor class which has behaviour of applying a discount to the original price.

```python
import logging

logging.basicConfig(level=logging.INFO)

class Discount: #1
    """Descriptor class to apply discount on prices."""
    def __init__(self, discount: float) -> None:
        self.discount = discount #2

    def __set_name__(self, owner, name): #3
        self.public_name = name 
        self.private_name = "_" + name

    def __get__(self, instance, owner):
        if instance is None: #4
            logging.info(f"Calling descriptor from class {owner}.")
            return self 
        logging.info(f"Fetching result from attribute {self.public_name}")
        original_price = getattr(instance, self.private_name) #5
        return original_price * self.discount

    def __set__(self, instance, value):
        logging.info(f"Setting {value=} for {self.public_name} on {instance.__class__.__name__}.")
        setattr(instance, self.private_name, value) #6

class PS5:
    """Company product."""
    black_friday_price = Discount(0.8) #7
    christmas_price = Discount(0.9) #8
    
    def __init__(self, price: float) -> None:
        self.price = price 
        self.black_friday_price = price 
        self.christmas_price = price 

class Xbox:
    """Company product."""
    black_friday_price = Discount(0.7)
    christmas_price = Discount(0.8)
    
    def __init__(self, price: float) -> None:
        self.price = price 
        self.black_friday_price = price 
        self.christmas_price = price 

if __name__ == "__main__":
    ps5 = PS5(500)
    print(ps5.black_friday_price)
    print(ps5.christmas_price)
```

1. Our descriptor class is defined as `Discount`
2. We can parametrise how much of a discount to apply by passing this into the constructor and have it accessible in the instance of the descriptors methods.
3. The `__set_name__` is a callback method, which allows the descriptor to have access to the class attribute name assigned to the descriptor in the client class. In this case, the attribute names such as `black_friday_price` and `christmas_price` in both the classes `PS5` and `Xbox`. This allows us to create two instance attributes, one public for logging and the other private for getting and setting data.
4. As mentioned before, the descriptor is assigned as a class attribute so in order to prevent `AttributeError` from occuring we need to handle the case where a user tries to call the descriptor on the client class rather than its instance.
5. Fetching the original price passed into the client class. Notice it does a lookup on the private name. This is because we will assign in the set, the value to the private name (rather than public name).
6. When the client class sets a attribute value to a class attribute which is assigned as a descriptor (example `black_friday_price`), this proxies a call to the `__set__` method of the descriptor. 
7. The descriptor `Discount` is now assigned to the `black_friday_price` class attribute. Anytime theirs a client attribute lookup or attribute setter like `ps5.black_friday_price` it will be invoking the `Discount` descriptor. We also parametrised the discount to have 20% off the price.
8. Similar to the above, but the discount is set to 10% off.

A careful reader might have noticed the issue with above if we wanted to change the original price such as:

```python
>>> ps5 = PS5(500)
>>> before = ps5.black_friday_price
>>> ps5.price = 400
>>> after = ps5.black_friday_price 
>>> assert before != after
```

Wouldn't update because we actually store a copy of the original price used inside the descriptor protocol within the constructor. 

```python
    def __init__(self, price: float) -> None:
        self.price = price 
        self.black_friday_price = price # calls descriptors __set__
        self.christmas_price = price # calls descriptors __set__
```

This data can get out of sync between the client and the descriptor. One possible way around this will be to provide a method in the client class as means to update both the original price 

```python
    def update_price(self, price: float) -> None:
        self.price = price 
        self.black_friday_price = price 
        self.christmas_price = price
```

A more abstract way of doing this to decouple the behaviour from the client is to have the descriptor be able to know which attribute to lookup in the client class to find out the original price. Now this completely eliminates the need to store the price data inside the descriptor with the added help of giving the descriptors constructor some information about where it should look for the price to calculate the discount.

```python
class Discount:

    def __init__(self, discount: float, price_attr: str) -> None:
        self.discount = discount
        self.price_attr = price_attr

    def __get__(self, instance, owner):
        if instance is None:
            logging.info(f"Calling descriptor from class {owner}.")
            return self 
        original_price = getattr(instance, self.price_attr)
        return original_price * self.discount


class PS5:

    black_friday_price = Discount(0.8, price_attr="price")
    christmas_price = Discount(0.9, price_attr="price")
    
    def __init__(self, price: float) -> None:
        self.price = price 


if __name__ == "__main__":
    ps5 = PS5(500)
    print(ps5.black_friday_price)
    print(ps5.christmas_price)
    ps5.price = 400
    print(ps5.black_friday_price)
    print(ps5.christmas_price)
```

Now we are back to a Non-data descriptor, which I think is easier to maintain. There will be some occations though you cannot avoid setting the value inside the descriptor itself.

# Summary

Descriptors are powerful OOP ways of abstracting some behaviour you would like to use across many client classes. However, it does require extra thought and can sometimes lead to over engineering a problem. So please make sure you've first explored simpler solutions such as `properties` before jumping into building a descriptor. Rule of thumb is if you want some dynamic computation in your class and its only for one class then properties/methods are your friends if you find yourself repeating the same boilerplate logic across many classes then explore descriptors.

Thanks for reading!

