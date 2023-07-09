---
layout: post
title: "Not all itertools are lazy"
tags: [python, iterators]
comments: false
---

Simple demonstration that [itertools.product] does not retrieve elements
from the input iterables lazily, i.e. only when the next element is needed.
Rather, the `product` object will read all values immediately upon its
construction.

This means that if iterators and/or generators are used as the input iterables,
all their elements will be consumed right after the call to `product`.

```python
import itertools

iterator = iter(['iced', 'hot'])
generator = (item for item in ['coffee', 'tea'])

menu = itertools.product(iterator, generator)
```

By now, both `iterator` and `generator` have been exhausted and will yield no
more values:

```pycon
>>> list(iterator)
[]
>>> list(generator)
[]
```

The `product` object will produce the pairs of values as expected:

```pycon
>>> for beverage in menu:
...     print(*beverage)
...
iced coffee
iced tea
hot coffee
hot tea
```

This behavior dates back to Python 2 but is only explicitly mentioned in
the official documentation since version 3.9:

> Before product() runs, it completely consumes the input iterables, keeping
> pools of values in memory to generate the products. Accordingly, it is only
> useful with finite inputs.

[itertools.product]: https://docs.python.org/3/library/itertools.html#itertools.product
