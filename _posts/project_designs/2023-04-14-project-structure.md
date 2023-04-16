---
layout: post
title: "Structuring your python project"
mermaid: true
categories: [project_design]
tags: [python,first_project]
---

Over the years, in the python community, you'll find many different flavours for setting up a project. Some favoured the flat layout for its practicality and others favoured the src layout for its saftey (more on that later). I'll give the pros and cons of each and explain why I prefer the later approach and typically use this whenever creating new projects at work or for personal use.

---

## Flat layout

As per defined on the [python docs](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/)

> The "flat layout" refers to organising a project's files in a folder or repository such that the various configuration files and import packages are all in the top-level directory.
{: .info}

An simple example of this might be

```
.
├── mypkg/
│   ├── __init__.py
│   └── my_module.py
├── tests/
│   ├── __init__.py
│   └── test_my_module.py
├── pyproject.toml
├── LICENCE
└── README.md
```

Our package is defined at the root folder level which contains a module called `my_module.py`. I've set in my `pyproject.toml` to install the source code located in `mypkg`. Lets discuss some of the concerns developers have had with this layout.

### Why should we avoid the flat layout?

One of the main issues with the flat lyaout is that tests can mysertiously even if your package build fails. Lets look at this a bit more closely.

```python
# my_module.py
def add(x, y):
    return x + y
```

with the following test

```python
# test_my_module.py
from mypkg.my_module import add
def test_add():
    assert add(4, 3) == 7
```

If we run the tests (without installing the packages) by doing

```sh
$ pytest tests/
```

the tests will pass (as they should). Lets try and break something here and show that running the same tests again fails to catch the breakage. I'm going to create a module in the root directory called `pain.py` and have it define some magic numbers. This module won't be shipped with the source distribution as it lives outside of `mypkg`. Hence, it shouldn't be available to the consumers of this package at runtime (and should break our projects package).

```python
# pain.py
SPECIAL_NUMBER = 3
```

Our project structure currently looks like this

```sh
.
├── pain.py
├── mypkg/
│   ├── __init__.py
│   └── my_module.py
├── tests/
│   ├── __init__.py
│   └── test_my_module.py
├── project.toml
├── LICENCE
└── README.md
```

Now back in `my_module.py` I will define a new function that uses this special number.

```python
# my_module.py
from pain import SPECIAL_NUMBER

def add(x, y):
    return x + y

def add_by_special_number(x):
    return add(x, SPECIAL_NUMBER)
```

If we go ahead and add a test for this in `test_my_module.py`

```python
# test_my_module.py
from mypkg.my_module import add
def test_add():
    assert add(4, 3) == 7

def test_add_by_special_number():
    assert add_by_special_number(1) == 4
```

And run the tests again (without installing mypkg) it passes. But the user is unaware we've silently broken the package by failing to package the project properly.

We can verify this by installing the package and then in a separate dir (not in the projects root folder) try importing `my_module` to notice its been broken

```sh
>>> from mypkg import my_module
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/sotirigeorgiou/.experimental/example_project/.venv/lib/python3.11/site-packages/mypkg/my_module.py", line 1, in <module>
    from pain import SPECIAL_NUMBER
ModuleNotFoundError: No module named 'pain'
```

This happens because python automatically adds your current directory in your `sys.path`. You can verify this by loading up the REPL 

```sh
>>> import sys
>>> sys.path
['', '/Users/sotirigeorgiou/.pyenv/versions/3.11.0/lib/python311.zip', '/Users/sotirigeorgiou/.pyenv/versions/3.11.0/lib/python3.11', '/Users/sotirigeorgiou/.pyenv/versions/3.11.0/lib/python3.11/lib-dynload', '/Users/sotirigeorgiou/.experimental/example_project/.venv/lib/python3.11/site-packages']
```

And you'll see the empty string `''` be appended into the path list. This is slightly dangerous because when running tests in your projects root folder, those modules will be discoverable even though they wont be packaged up in your source distribution. This leads to confusion as to why your tests were passing even though our package had broken imports.

> Separating your source distribution forces you to install the package before testing, leading to more reliable behaviours for your end users.
{: .prompt-tip}

---

## Src layout (recommended)

Aside from the [python docs](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/) generally recommending this layout, I want to see how we overcome the issue in the last example by following this layout.

We first modify the `pyproject.toml` to point to the `src` folder to look for packages to install as part of our source distribution.

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"

[project]
name = "mypkg"
version = "0.0.1"

[tool.setuptools]
package-dir = {"" = "src"}
packages = ["mypkg"]
```

> For python version >= 3.11, setuptools has automatic package discovery for src layout. 
{: .prompt-info}

Now lets use the `src/` and move our boilerplate code in there as such.

```shell
.
├── src/ 
│   ├── mypkg/
│   │   ├── __init__.py
│   │   └── my_module.py
│   └── pain.py
├── tests/
│   ├── __init__.py
│   └── test_my_module.py
├── project.toml
├── LICENCE
└── README.md
```

Now to verify that we've broken our installation through our tests we can use [tox](https://tox.wiki/en/latest/user_guide.html) (or any other tool that provides a separate venv to build and test our source distribution). 

```
$ tox -r -e py11

=============================================================== ERRORS ===============================================================
______________________________________________ ERROR collecting tests/test_my_module.py ______________________________________________
ImportError while importing test module '/Users/sotirigeorgiou/.experimental/example_project/tests/test_my_module.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
../../.pyenv/versions/3.11.0/lib/python3.11/importlib/__init__.py:126: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
tests/test_my_module.py:1: in <module>
    from mypkg.my_module import add, add_by_special_number
.tox/py311/lib/python3.11/site-packages/mypkg/__init__.py:1: in <module>
    from pain import SPECIAL_NUMBER
E   ModuleNotFoundError: No module named 'pain'
```

Perfect, we've now captured our error during our tests!

Some of the main benefits of the `src` layout is

* Relies on the installation of the project in order to run its code. This allows you to capture changes that breaks your installation (such as import errors).
* Provides a clear separation between source code and other project metadata when building your source distribution.
* Mitigates the reliance of the current working directory being added to sys.path which can incorrectly cause tests to pass.


One of the main bottlenecks associated the the `src` layout is having to manage another nested directory at the root directory. But I beleive the pros outway the cons for this.