---
layout: post
title:  "Positional-only and keyword-only parameters"
tags: [python]
comments: false
---

An overview of some neat peculiarities of the positional-only and
keyword-only parameters in Python.

## Positional-only parameters

The main and the most obvious feature of the positional-only parameters
is that unlike regular positional parameters, their values cannot be
passed as named arguments.

```python
def add(x, /, y):
    return x + y
```

```pycon
>>> add(1, 2)
3
>>> add(x=1, y=2)
...
TypeError: add() got some positional-only arguments passed as keyword arguments: 'x'
```

Unlike regular positional parameters, positional-only parameters can
have a default value, in which case they are not required.

```python
def add(x, y=0, /):
    return x + y
```

```pycon
>>> add(1, 2)
3
>>> add(1)
1
```

Note that you still must observe the basic syntactic rule requiring that
in a function definition, a parameter with a default value cannot be
followed by a parameter without a default value.

```pycon
>> def add(x=0, y, /):
...     return x + y
... 
SyntaxError: non-default argument follows default argument
```

## Keyword-only parameters

The main and the most obvious feature of the keyword-only parameters
is that unlike regular keyword parameters, their values cannot be
passed as positional arguments.

```python
def add(x, *, y=0):
    return x + y
```

```pycon
>>> add(1)
1
>>> add(1, 2)
...
TypeError: add() takes 1 positional argument but 2 were given
```

Unlike regular keyword parameters, keyword-only parameters don't need
to have a default value, in which case they are required.

```python
def add(x, *, y):
    return x + y
```

```pycon
>>> add(1, y=2)
3
>>> add(1)
...
TypeError: add() missing 1 required keyword-only argument: 'y'
```

Interestingly, within the group of keyword-only parameters in a function
definition, a parameter with a default value can even be followed by
a parameter without a default value.

```python
def add(x, *, y=0, z):
    return x + y + z
```

```pycon
>>> add(1, y=2, z=3)
6
>>> add(1, z=3)
4
```
