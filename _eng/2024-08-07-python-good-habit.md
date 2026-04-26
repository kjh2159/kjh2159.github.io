---
title: (Python) Good Coding Habits Part 1
date: 2024-08-07 07:00:00 +0900
categories: [python, habit]
tags: [python, habit, coding, programming]
---

# Good Coding Habits in Python Part 1

I plan to share a series of useful habits that can help you write better Python code. The habit introduced in this post is `Type Hint`.

## Introduction

What is a type hint? As the name suggests, a type hint gives a hint about the type of a variable, argument, or return value. Unlike `C/C++`, where data types are statically defined, Python allows the type of a variable to change dynamically. This characteristic has very clear advantages and disadvantages.

The advantage is flexibility. For example, if the result of a certain processing step has a completely different data type, you do not necessarily need to declare a new variable for it. Of course, depending on the situation, you may still need to preserve the original data.

The disadvantage is that the code can become harder to understand. When tracing or analyzing code, you may use tools such as PyCharm, Anaconda, or VS Code extensions to perform syntax checking and run background analysis. However, these tools are not perfect. If the type of a variable keeps changing, or if multiple types can be assigned to the same variable, these tools may not be able to trace the code accurately. These features are often related to syntax linting, type resolution, and code navigation.

In Python, one useful way to address this issue is `Type Hint`.

## Syntax: Initialization

The basic structure is as follows.

```python
var_name: type = value
```

The important part here is `: [type]`. This allows you to provide a hint about the type of the variable. Below is a simple example.

```python
var: int = 10
```

The syntax itself is very simple. However, if you are not used to it, you may accidentally write the variable name and type in the wrong order. Since my main languages are `C/C++`, I sometimes confused this with `int var = 10;`. Other than that, the syntax is quite straightforward.

## Syntax: Function Arguments

Type hints can also be used for function arguments. The purpose is the same.

```python
def function_name(var_name: type = value) -> return_type:
    # do something
    pass
```

The default value and return type are optional. It may be hard to understand just by looking at the general function form, so let us look at a simple example.

```python
def foo(a: int, b: int):
    print(f'a = {a} / b = {b}')
```

An example with optional parts can be written as follows.

```python
def foo(a: int = 5) -> None:
    print(f'a = {a}')

foo()
foo(1)
```

This function can be interpreted as follows. The argument `a` of the function `foo` has a default value of `5`, and the function returns `None`.

## Type Hint Support

### Multiple Type Hints

In the previous examples, it may look as if only one type can be used for `type`. However, in practice, multiple types can also be specified. This can be done using the `|` operator.

Of course, the best case is when only one type is allowed. However, there are situations where multiple types must be supported. The problem with allowing multiple types is also clear. For example, suppose a variable can be either `int` or `list`. A `list` has the `append` method, but an `int` does not. In many cases, a code linter may still show that this variable has an `append` method. If you accidentally call this method on an integer, Python will naturally raise an `AttributeError`.

The following code is an example of giving hints for multiple types.

```python
def foo(a: int | float | None, 
        b: int | float | None) -> None:
    print(a + b)
```

### [typing] Union

The `|` operator is not the only feature that supports type hints. Since Python 3.5, Python has provided the `typing` module to support type hinting. Inside this module, `Union` provides functionality similar to `|`.

The syntax of `Union` is as follows.

```python
Union[type, ...]
```

`Union` groups several possible types into one type expression. This can improve readability when type hints become long. Below is an example using `Union`.

```python
from typing import Union

def foo(a: Union[int, float, None]) -> None:
    if not a:
        return
    print(a)
```

In this code, if `a` is `None`, the function ends without printing anything. Otherwise, it prints `a`. When using `Union` from the `typing` module, you must import it first.

### [typing] Optional

If you have looked at the examples above, you may have noticed that `None` often appears at the end of type hints. This kind of expression is used very frequently in real-world development.

For example, when using database frameworks such as `MySQL` or `SQLite`, a `SELECT` operation may return `None` if no matching result is found. The `typing` module also provides a class for this situation.

The syntax is as follows.

```python
Optional[type]
```

This means `type | None`.

```python
from typing import Optional, Union

def foo(a: Optional[Union[int, float]]) -> None:
    if not a:
        return
    print(a)
```

In the past, I did not feel that this style was used very often. However, when looking at recently released modules, I feel that the `typing` module is being used much more frequently. I think this is because type hints are helpful for writing precise code and improving refactoring.

### [typing] Final

`Final` is a special type hint. It tells the type checker that a variable is final. This means that a variable declared with `Final` should not be reassigned or overridden.

The syntax is as follows.

```python
Final[type]
```

It is very easy to use. If the type is obvious or not important to specify explicitly, you can simply write `Final`. Below are two examples that official documentation treats as errors reported by type checkers.

```python
from typing import Final

MAX_SIZE: Final = 9000
MAX_SIZE += 1  # Error reported by type checker
```

```python
from typing import Final

class Connection:
    TIMEOUT: Final[int] = 10

class FastConnector(Connection):
    TIMEOUT = 1  # Error reported by type checker
```

The first example represents reassignment of a variable, while the second represents overriding a class variable. In other words, `Final` is similar to the `const` keyword in `C++`, in the sense that the value or type should not be changed.

### [typing] Callable

`Callable` is a special type hint that represents something that can be called. A `Callable` can be a function, a class constructor, or any other object that can be invoked.

The syntax is as follows.

```python
Callable[[arg_type, ...], return_type]
```

The types inside the inner list represent the argument types passed when the object is called. The type after the comma represents the return type.

Using `Callable` itself is quite simple. However, one thing to be careful about is that `typing.Callable` is marked as deprecated in the official documentation. The documentation describes it as a “deprecated alias to collections.abc.Callable.” In other words, the actual definition of `Callable` is in `collections.abc`, so it is preferable to import it from there.

Below is an example.

```python
from collections.abc import Callable

def foo(a: int = 1) -> int:
    return a
    
def bar(func: Callable[[int], int], var: int) -> None:
    print(func(var))
```

This example may look slightly tricky, but the code itself is simple enough to understand. In this way, Python can explicitly express functions similar to `forEach`, which is commonly used in JavaScript.

If you still want to manage this under the `typing` style, you can write it as follows. However, this approach is a bit more tricky.

```python
import collections
from typing import Callable

Callable = collections.abc.Callable
```

## Generic Types

I am not entirely sure what the best Korean translation of “generic type” is, so I will simply call it a generic type here.

### List

Python supports generic types as type hints to improve code flexibility, maintainability, and code linting. As the name suggests, generic types allow us to describe existing Python data structures in a more detailed way.

Of course, many people may think that simply writing `list` is enough, as shown below.

```python
a: list = [1, 2, 3]
```

That is also possible. However, this does not clearly describe the types of the elements inside the list. In such cases, you can write the type hint as follows.

```python
from typing import List

a: List[int] = [1, 2, 3]
```

By writing it this way, code linters can analyze even more complex code more flexibly and accurately.

### Dictionary

The `typing` module also supports dictionary types. The syntax is as follows.

```python
Dict[key_type, val_type]
```

Using this format, we can write code like the following.

```python
from typing import Dict, List

scores: Dict[int, List[str]] = {
    101010: ["TOM", "B"],
    987654: ["HARRISON", "C"]
} 
```

### Others

There are many other generic types as well. Commonly used ones include `typing.Tuple`, `typing.Set`, `typing.Iterable`, `typing.Literal`, and `typing.Any`. Of course, whether `Iterable` and `Literal` should be called generic types may be somewhat debatable.

# Closing

As the name suggests, a type **hint** is only a hint. Therefore, Python does not raise an error just because a type hint is incorrect, as long as the syntax itself is valid. Of course, if you use type hints incorrectly, it will naturally become harder to write, understand, and analyze your own code. Code linters may also show warnings.

This post ends here, but the features introduced here are not everything. Python provides many other type hinting features as well. Still, I have covered some of the most essential ones. I hope these features help you write better code.