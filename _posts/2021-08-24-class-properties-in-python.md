---
layout: post
title:  "Class Properties in Python"
tags: [python, oop]
comments: false
---

As of Python version 3.9, it is possible to define a class property (i.e. a
computed or otherwise managed class attribute) by simply stacking
the `classmethod` and `property` decorators.

For example:

```python
class Class:
    @classmethod
    @property
    def reversed_mro(cls):
        return cls.mro()[::-1]


class SubClass(Class):
    ...
```

Accessing the attribute for reading works as expected in Python 3.9:

```pycon
>>> # via class
>>> Class.reversed_mro
[<class 'object'>, <class '__main__.Class'>]
>>> SubClass.reversed_mro
[<class 'object'>, <class '__main__.Class'>, <class '__main__.SubClass'>]

>>> # via instance
>>> Class().reversed_mro
[<class 'object'>, <class '__main__.Class'>]
>>> SubClass().reversed_mro
[<class 'object'>, <class '__main__.Class'>, <class '__main__.SubClass'>]
```

as opposed to version 3.8 or earlier:

```pycon
>>> Class.reversed_mro
<bound method ? of <class '__main__.Class'>>
>>> SubClass.reversed_mro
<bound method ? of <class '__main__.SubClass'>>
```

This change to the language was introduced as the result of [this issue][issue],
with class properties being mentioned in the discussion as one of the potential
benefits of it.

However, carrying on with the above example we will soon discover something
interesting: it is possible to set and delete the property attribute without any
restrictions:

```pycon
>>> Class.reversed_mro = []
>>> Class.reversed_mro
[]
>>> del Class.reversed_mro
>>> hasattr(Class, 'reversed_mro')
False
```

This is not consistent with the behaviour of a regular instance property whose
`setter` and `deleter` methods are not explicitly defined. For example:

```python
class Class:
    @property
    def reversed_mro(self):
        return type(self).mro()[::-1]
```

```pycon
>>> instance = Class()
>>> instance.reversed_mro
[<class 'object'>, <class '__main__.Class'>]
>>> instance.reversed_mro = []
AttributeError: can't set attribute
>>> del instance.reversed_mro
AttributeError: can't delete attribute
```

It is unclear to me whether this inconsistency was foreseen and/or intended by
the Python core developers.
[This unit test][test] was added to Python's test suite as part of the change,
but it is just a simple test that doesn't attempt to set or delete the
attribute. It seems that this part of behaviour of a class property
(i.e. the setting and deleting part of its descriptor protocol) may have not
been taken into account at all.

## Update

In the *What’s New In Python 3.11* document, the possibility of decorating
`property` with `classmethod` is announced to be [deprecated][deprecation].

> Chaining `classmethod` descriptors (introduced in bpo-19072) is now deprecated.
  It can no longer be used to wrap other descriptors such as `property`.
  The core design of this feature was flawed and caused a number of downstream problems.

[issue]: https://github.com/python/cpython/issues/63272
[test]: https://github.com/python/cpython/commit/805f8f9afea116c5d4d000570e3d02ae84502f43#diff-510a022afde6dbb437080870cced7548f338fb8654a4df10c425e5105a83b2e3
[deprecation]: https://docs.python.org/3.11/whatsnew/3.11.html#deprecated
