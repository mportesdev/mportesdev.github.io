---
layout: post
title:  "Extending a Path in Python"
date:   2021-09-15
---

This article demonstrates the pitfalls related to subclassing and extending the
[`pathlib.Path`][path] class.

Let's suppose we work with files and directories in Python and we would like
to extend `Path` with some cool extra feature. Let's say we want to overload
the `<<` operator, so instead of e.g. `path.parent.parent.parent` we can write
just `path << 3`.

## Subclassing Path

Our custom class will simply inherit from `Path` and the corresponding special
method will be implemented in a simple recursive way:

```python
from pathlib import Path


class CoolPath(Path):
    def __lshift__(self, other):
        if other == 0:
            return self
        return self.parent << other - 1
```

First thing we learn is that subclassing Path the usual way does not work.

```pycon
>>> CoolPath('/home/michal/python/projects/')
AttributeError: type object 'CoolPath' has no attribute '_flavour'
```

The explanation of this error is that the `Path.__new__` method is
somewhat specific: it does not actually return objects of the `Path` class.
Rather, it serves as a dispatch mechanism to choose between two types of
path objects, depending on the current operating system.

From the `pathlib` [source code][gh]:

```python
    def __new__(cls, *args, **kwargs):
        if cls is Path:
            cls = WindowsPath if os.name == 'nt' else PosixPath

        ...
```

What causes trouble here is the `if cls is Path` hardcoded test that makes it
impossible for potential subclasses of `Path` to have instances successfully
constructed.

For example, when invoked via our custom subclass, the `cls`
argument will be `CoolPath` rather than `Path`. This means that the
platform-testing conditional expression will be skipped and we will get the
above shown flavour-related error.

We should be able to overcome this constraint by passing `Path` explicitly
as the `cls` argument:

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
```

```pycon
>>> path = CoolPath('/home/michal/python/projects/')
```

Good news is that an object has been constructed successfully without the
previously seen error. But does it do what we want it to?

```pycon
>>> path << 1
TypeError: unsupported operand type(s) for <<: 'PosixPath' and 'int'
```

No, it does not. The problem here is that the object is now in no way related
to `CoolPath` and does not have any knowledge of the overridden
`__lshift__` method.

## Subclassing platform-aware path classes

It seems that a possible solution to our subtyping headache should be to mimic
the dispatch conditional we have seen in the source code of `Path.__new__`,
and instead of subclassing `Path`, use one of the platform-specific classes
as our base class. Then we should be able to inherit from this base class.

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
```

Let's take our custom class for a test ride:

```pycon
>>> path = CoolPath('/home/michal/python/projects/')
>>> path << 1
CoolPath('/home/michal/python')
>>> assert path << 0 == path
>>> assert path << 2 == path.parent.parent == path.home()
>>> path << 100
CoolPath('/')
```

Great, everything works as intended.

## Final note

It seems likely that the `Path.__new__` method is written in such a way
that it does not allow for direct subtyping of the `Path` class.
It is my understanding that the `if cls is Path` hardcoded condition is there
to skip the dispatch when it is not necessary, i.e. when a `WindowsPath()` or
`PosixPath()` call is made.

Thank you for reading, and as usual, feel free to contact me if you think
something is wrong or missing in this blog post.

[path]: https://docs.python.org/3/library/pathlib.html#pathlib.Path
[gh]: https://github.com/python/cpython/blob/v3.9.7/Lib/pathlib.py#L1079
