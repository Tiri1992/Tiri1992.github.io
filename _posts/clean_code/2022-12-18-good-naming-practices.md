---
layout: post
title: "Clean code - Pythonic naming conventions"
mermaid: true
categories: [clean_code,good_practices]
tags: [python,clean_code,pythonic]
---

Naming variables, functions and classes is one of the critical areas of clean code practices. We've all been in the situation where we are new to a codebase and have come across functions that are so poorly named we scratch our heads and wonder what is it even doing? This complexity can be detrimental to a development team because the developer now needs to go dig deeper into the logic of that function to try figure out what it does (and godforbid comes across another vaguely named function) ends up wasting a lot of mental energy and time.

In this tutorial I'm going to demonstrate some examples of `bad`, `okay` and `good` naming conventions. We can finish off with refactoring a sample of code thats poorly named and use some of the practices we've learnt to make it more readable, clean and pythonic.

---
## Variable names

We should always try to be as explicit as possible without introducing redundancies. Typically, we need to have an idea in mind of what types they hold, i.e. strings, bool, container types, ints etc. This will sometimes guide our choice of names.


Lets look at some examples of variables which are of type `bool` and hold information about a user:
```python
# Bad 
finished = True

# Okay
has_activated_user = True

# Good
is_active = True
```

The distictions here are:
* Bad example, does not tell us any information as to what has been finished. Its extremely vague.
* Okay example, tells us that a user has been activated but has some redundant information like mentioning the word user even though its already in that domain
* Good example, is nice for two reasons. Its explicit in what the variable represents and its also "truthy" i.e. its name is positive rather than `is_not_active=False` (takes an extra second to think about that right!)

Lets have a look at an example where we want to hold some container data like a `list`. The goal here is to use nouns of short phrases with adjectives that describe what type of data the container holds i.e. is it user data or logs data etc?

```python
# Bad
stuff = ["Jane", "Michael", "Alex"]

# Okay
people = ["Jane", "Michael", "Alex"]

# Good
user_names = ["Jane", "Michael", "Alex"]
```

The distinctions here are:
* Bad example, does not provide any information to the developer about the type of data this container holds.
* Okay example, improves on this that they are people but its still not explicit as to describe how they relate to the wider program
* Good example, provides us with an explicit description that these are user names in our application.

---
## Function/Method names

Lets move on to discussing how we could improve on naming our functions and methods so they as explicit and describe the behaviour they perform. Typically, to conform with [PEP8 standards](https://peps.python.org/pep-0008/), our function/methods will use `snake_case`. Before we go any further, I would like to explain that naming functions/methods are divided into two camps:

1. Functions that perform an operation
2. Functions that compute a boolean

Lets jump into some examples for (1), suppose we wanted to save user data to a database:

```python
# Bad
def process():
    ...

# Okay
def save():
    ...

# Good
def save_user():
    ...
```

The distinctions here are:
* Bad example, does not tell us what is being processed or whether process is part of a larger series of operations to save a user to a database.
* Okay example, it tells us *what* the operation is but it doesn't describe what its operating on.
* Good example, extends the okay example to now tell us the oprand (i.e. what the operator is acting on), in this case saving a user.

Before we move on, I just want to reiterate that the *Okay* example might have been fine if the method was bounded to a class which described the actor i.e.

```python
# Good
class User:

    def save(self):
        ...
```

Lets have a look at some examples for (2). With functions/methods that return `booleans` we should be as explicit and "truthy" as possible without introducing redundancy. Suppose we are writing a function which should validate the users input:

```python
# Bad
def process():
    ...

def do_it():
    ...

# Okay
def check():
    ...

def validate_save():
    ...

# Good

def is_valid():
    ...
```

The distinctions here are:
* Bad example, again is so vague developer will almost certainly have to look at the internal logic to figure out what its doing (sign of code smell).
* Okay example, `check()` tells us what its doing while being vague on what its exactly checking i.e. if its valid. On the other hand, `validate_save()` misleads us to thinking its doing both operation.
* Good example, its "truthy" hence returns a boolean (think of these truthy names as being question-like) and also telling us what its doing i.e. validation.

---
## Class names

When naming classes, we should typically use nouns to describe the class. These will be instaniated as object hence words like `User`, `Product`, `Store` are good class names. We can always be a bit more explicit on the instance names. Also, avoid redundant suffixes like `DatabaseManager`, this raises more questions than answers.

Lets look at some examples, where we want to create a `User` like class:

```python
# Bad

class UEntity:
    pass 

class ObjA:
    pass

# Okay

class UserObj:
    pass 

class AppUser:
    pass

# Good
class User:
    pass

class Admin:
    pass
```

The distinctions here are:
* Bad example, does not describe the class clearly as to what it represents.
* Okay examples, Slightly better but we have some redundant info such as `UserObj` where we already know that class will be instaniated as an object. `AppUser` is also redundant as every user is part of the application.
* Good examples, these are very explicit. Even more so with `Admin`, as this tells us what type of user class we are creating.

---
## Being overtly descriptive

I want to provide a couple of examples of when being too explicit can either be redundant and create more confusion when naming. Lets consider a basic `User` class and instaniate it with a name that highlights this point:

```python
class User:
    def __init__(self, name: str, age: int) -> None:
        self.name = name 
        self.age = age 

user_with_name_and_age = User(name="Alex", age=30)
```

We don't need to describe the property names of the `user` object because thats already provided inside the signature. Another naming issue I've come across in the wild is where the method names sound oh so similar that I've glanced at the class methods and started scratching my head. Heres an example:

```python
# Suppose we have an analytics object
analytics.get_daily_data(day)
analytics.get_day_data()
analytics.get_raw_daily_data(day)
analytics.get_parsed_daily_data(day)
```

This is very cofusing because these methods sounds so similar. We should try to choose specific names that perhaps describe the type of data they are returning. Either way just try to be specifc. So altering the above might become:

```python
analytics.get_daily_report(day)
analytics.get_data_for_today()
analytics.get_raw_daily_data(day)
analytics.get_parsed_daily_data(day)
```

---
## Refactoring example

Lets try and end what we've learnt by refactoring some code which has a few bad naming conventions and attempt to use what we've learnt to improve it.

```python
class Point:
    def __init__(self, coordX, coordY):
        self.coordX = coordX
        self.coordY = coordY


class Rectangle:
    def __init__(self, starting_point, broad, high):
        self.starting_point = starting_point
        self.broad = broad
        self.high = high

    def area(self):
        return self.broad * self.high

    def end_points(self):
        top_right = self.starting_point.coordX + self.broad
        bottom_left = self.starting_point.coordY + self.high
        print('Starting Point (X)): ' + str(self.starting_point.coordX))
        print('Starting Point (Y)): ' + str(self.starting_point.coordY))
        print('End Point X-Axis (Top Right): ' + str(top_right))
        print('End Point Y-Axis (Bottom Left): ' + str(bottom_left))


def build_stuff():
    main_point = Point(50, 100)
    rect = Rectangle(main_point, 90, 10)

    return rect


my_rect = build_stuff()

print(my_rect.area())
my_rect.end_points()
```

Lets start by looking at the `Point` class, theres redundant information here in the signature: `coordX`, `coordY`. We already know that this class contains `x` and `y` coordinates and I think this is specific enough.

```python
# Refactored Point class
class Point:
    def __init__(self, x: int, y: int) -> None:
        self.x = x 
        self.y = y
    
    def __repr__(self) -> str:
        return f"{self.__class__.__name__}({self.x}, {self.y})"

```

I've also added the `__repr__` dunder method so that we can have the coordinates rendered when printing to stdout. Next, lets work on refactoring the `Rectangle` class. I've gone with renaming the `starting_point` as `bottom_left_point` to be explicit about where the starting point should be. This helps map out the remaining points when we print these out. However, this could have also been done in several other ways such as using `origin` as a name or picking any other corner. Furthermore, notice how vague `broad` and `high` was as names, I mean if your WTF alarm is going off right now its totally understandable. Simple is better here and `width`, `height` are better names.

```python
class Rectangle:

    def __init__(self, bottom_left_point: Point, width: int, height: int) -> None:
        self.bottom_left_point = bottom_left_point
        self.width = width
        self.height = height

    def get_area(self) -> int:
        return self.width * self.height

    def print_coordinates(self) -> None:
        top_left = Point(x=self.bottom_left_point.x, y=self.bottom_left_point.y + self.height)
        bottom_right = Point(x=self.bottom_left_point.x + self.width, y=self.bottom_left_point.y)
        top_right = Point(x=self.bottom_left_point.x + self.width, y=self.bottom_left_point.y + self.height)

        print(f"Bottom Left: {self.bottom_left}")
        print(f"Top Left: {top_left}")
        print(f"Top Right: {top_right}")
        print(f"Bottom Right: {bottom_right}")
```

To cover the remaining changes, I've allocated for a verb when computing `area`. This is better convention as `area` is not a property but a method that the client would like to recieve some value for. I'm now recreating the points based off the location of the bottom left and then leveraging the `__repr__` method of the point objects to print their positions to console. This I think is much more explicit and is using the position of the `bottom_left_point` to create the other points based off the `width` and `height` provided. I also changed the `end_points()` method because we could be even more explicit and describe what we are doing (i.e. printing to console) and what type of data we are printing (i.e. coordinates), so `print_coordinates` seemed like a better name.

Now over to the remaining functions. If I wrote `build_stuff()` on a PR for sure my team mates would murder me (rightly so!). This doesn't give me any information about what its building (again being vague). I've refactored it as so:

```python
def create_rectangle(bottom_left_point: Point, width: int, height: int) -> Rectangle:
    return Rectangle(
        bottom_left_point=bottom_left_point,
        width=width,
        height=height,
    )
```

Using the name `create_rectangle` we know we have a factory like method here, which is explicit. By doing this we can pass in all the necessary arguments to create the `rectangle` object on the fly. I've also used a refactoring technique called `inline` refactor, where by we move the squash the operations to a single line (i.e. the initialisation of `rectangle` object). We will speak more about refactoring techniques in future tutorials.

The remaining parts of the code now become:

```python
rectangle = create_rectangle(
    bottom_left_point=Point(50, 100),
    width=90,
    height=10,
)

print(rectangle.get_area())
print(rectangle.print_coordinates())
```

Hope you've learnt something new about good naming practices in python, see you in a future tutorial!