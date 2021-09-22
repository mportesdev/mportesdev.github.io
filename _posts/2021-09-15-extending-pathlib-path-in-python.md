---
layout: post
title:  "Extending pathlib.Path in Python"
date:   2021-09-15
tags: [python, oop]
comments: false
---

This article attempts to summarize the _gotcha_ associated with subclassing
the [`pathlib.Path`][path] class from Python's standard library.

Let's suppose we work with files and directories in Python and we would like
to extend `Path` with some cool extra functionality. As a toy example,
let's say we want to overload the `<<` operator, so that instead of
e.g. `path.parent.parent.parent` we can just write `path << 3`.

## Subclassing Path

Unaware of any pitfalls, we simply define a custom class that inherits
from `Path`, and implement the corresponding special method:

```python
from pathlib import Path


class CoolPath(Path):
    def __lshift__(self, other):
        if other == 0:
            return self
        return self.parent << other - 1
```

We find out very soon that subclassing `Path` the usual way does not work.

```pycon
>>> CoolPath('/home/michal/python/scratches/')
AttributeError: type object 'CoolPath' has no attribute '_flavour'
```

To understand this error, it is necessary to take a look at what is going on in
the [`Path.__new__`][gh] method.

```python
def __new__(cls, *args, **kwargs):
        if cls is Path:
            cls = WindowsPath if os.name == 'nt' else PosixPath
        self = cls._from_parts(args, init=False)
        if not self._flavour.is_supported:
            raise NotImplementedError("cannot instantiate %r on your system"
                                      % (cls.__name__,))
        self._init()
        return self
```

As we can see, this constructor method is somewhat specific:
it does not actually return objects of its own class.
Rather, it serves as a dispatch mechanism to choose between two
types of path objects, depending on the operating system the Python
code is being run on.

It is my understanding that the `if cls is Path` test
is there to skip the dispatch in cases when it is not necessary,
i.e. when a `WindowsPath()` or `PosixPath()` call is made.

However, what causes trouble here is that `Path` is hardcoded in this test,
which makes it impossible for potential subclasses of `Path` to have
instances successfully constructed.
For example, when invoked via our custom subclass, the `cls` argument
will reference `CoolPath` rather than `Path`.
As a result, the reassignment of `cls` will be skipped and two lines later,
we will be faced with the above shown flavour-related error.

What can we do about it? We might try to overcome this constraint by
defining our own `__new__` method and passing `Path` explicitly as
the `cls` argument to the superclass:

```python
from pathlib import Path


class CoolPath(Path):
    def __new__(cls, *args, **kwargs):
        # force `Path` as the first argument to `Path.__new__`
        return super().__new__(Path, *args, **kwargs)

    def __lshift__(self, other):
        if other == 0:
            return self
        return self.parent << other - 1


path = CoolPath('/home/michal/python/scratches/')
```

Good news is that an object has been created successfully without the
previously seen error. But does it do what we want it to?

```pycon
>>> path << 1
TypeError: unsupported operand type(s) for <<: 'PosixPath' and 'int'
```

No, it does not. The problem here is that the object is now in no way related
to `CoolPath` and does not have any knowledge of the custom `__lshift__` method.

Apparently, a different approach must be taken.

## Subclassing platform-aware Path class

It seems that the definitive solution to our problem is to mimic
the dispatch conditional we have seen in `Path.__new__`,
and instead of subclassing `Path`, use one of the platform-specific classes
as a base class.

```python
import os

if os.name == 'nt':
    from pathlib import WindowsPath as PathBase
else:
    from pathlib import PosixPath as PathBase


class CoolPath(PathBase):
    def __lshift__(self, other):
        if other == 0:
            return self
        return self.parent << other - 1


path = CoolPath('/home/michal/python/scratches/')
```

An object has been created without an error. Let's take it for a test ride:

```pycon
>>> path << 1
CoolPath('/home/michal/python')
>>> assert path << 0 == path
>>> assert path << 2 == path.parent.parent == path.home()
>>> path << 100
CoolPath('/')
```

The overloaded operator works exactly as intended.

## Final note

I came to a conclusion that direct subtyping of `Path` is not possible due to
the way its `__new__` method is implemented.

Thank you for reading, and feel free to contact me if you know some tricks to
subclass `Path` directly or if you think something is wrong or missing
in this blog post.

[path]: https://docs.python.org/3/library/pathlib.html#pathlib.Path
[gh]: https://github.com/python/cpython/blob/v3.9.7/Lib/pathlib.py#L1079
