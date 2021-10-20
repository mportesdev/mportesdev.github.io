---
layout: post
title:  "Storing Reference to Save Method Look-up"
tags: [python, timing, oop]
comments: false
---

This is a simple timing experiment to explore whether it is possible
in Python to gain performance benefit by calling a method via a direct
reference instead of accessing it as an object's attribute.

Let's suppose we have a loop that contains a call to an object's method:

```python
for _ in range(n):
    ...
    obj.method()
    ...
```

In theory, if `n` is a large number, it should make sense to refactor the loop
like this:

```python
obj_method = obj.method

for _ in range(n):
    ...
    obj_method()
    ...
```

i.e. to store a reference to the method and then call it directly using this
reference as a shortcut. This technique should save us __n__ attribute look-ups
(method resolutions) which should result in reduced execution time.

Let's verify this assumption. To minimize overhead, we will define and call
an empty method. The time taken will be measured by the
[`timeit.timeit`][timeit] function.

```python
from timeit import timeit


class Class:
    def method(self):
        pass


obj = Class()

lookup_time = timeit('obj.method()', number=10_000_000, globals=vars())
print(f'Look-up: {lookup_time:.3f} s')

obj_method = obj.method    # store reference to method

shortcut_time = timeit('obj_method()', number=10_000_000, globals=vars())
print(f'Shortcut: {shortcut_time:.3f} s')
```

Output of three sample runs of the program:

```
Look-up: 1.148 s
Shortcut: 1.036 s
```

```
Look-up: 1.173 s
Shortcut: 1.107 s
```

```
Look-up: 1.150 s
Shortcut: 1.022 s
```

We can see that there is indeed a consistent difference (around 9%) in
execution time.

One thing to note here is that instance methods in Python are not resolved by
the means of simple attribute look-up. Rather, a descriptor protocol's getter
mechanism is used to retrieve bound methods. I assume that this adds
another tiny amount of overhead that is incurred during method calls
and that our shortcut saves us. Quoting the [official documentation][docs]:

> Note that the transformation from function object to instance method
> object happens each time the attribute is retrieved from the instance.
> In some cases, a fruitful optimization is to assign the attribute to
> a local variable and call that local variable.

## Final note

This shortcut trick should probably be used sparsely, as I think its
use cases are quite limited. It would perhaps make sense in some
critical loop with many method calls, and only in cases when the
loop represents an actual bottleneck in the program.

Thank you for reading, and feel free to contact me if you think something
is wrong or missing in this blog post.

[timeit]: https://docs.python.org/3/library/timeit.html#timeit.timeit
[docs]: https://docs.python.org/3/reference/datamodel.html#the-standard-type-hierarchy
