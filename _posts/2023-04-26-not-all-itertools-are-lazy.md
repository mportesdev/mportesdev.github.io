---
layout: post
title: Not all itertools are lazy
tags: [python, iterators]
comments: false
---

Simple demonstration that [itertools.product] does not retrieve values from the
input iterables lazily, i.e. only when needed. Rather, the `product` object
will read all values immediately upon its construction.

This means that if iterators/generators are used as the input iterables, all
their values will be consumed right after the call to `product`.

```python
import itertools

iter_1 = iter(['iced', 'hot'])
iter_2 = iter(['coffee', 'tea'])

menu = itertools.product(iter_1, iter_2)
```

By now, both `iter_1` and `iter_2` have been exhausted and will yield no more
values.

```pycon
>>> list(iter_1)
[]
>>> list(iter_2)
[]
```

The `product` object will produce the pairs as expected:

```pycon
>>> for variant, beverage in menu:
...     print(variant, beverage)
...
iced coffee
iced tea
hot coffee
hot tea
```

This behavior is explicitly mentioned in the official docs since Python
version 3.9:

> Before product() runs, it completely consumes the input iterables, keeping
> pools of values in memory to generate the products. Accordingly, it is only
> useful with finite inputs.

[itertools.product]: https://docs.python.org/3/library/itertools.html#itertools.product
