---
layout: post
title:  "Joining sequences in Python"
date:   2021-08-10
categories: articles
---

In this short article I would like to explore some of the numerous ways Python
offers to concatenate sequences such as lists, tuples, byte arrays etc.

## Joining sequences of different types

Let's assume we have the following data:

```python
data = [[1, 2], ('three', 'four'), bytearray([53, 54])]
```

Now we want to join all the iterable sequences stored in `data` together into
a single list. In other words, we want to flatten `data`.

**Solution 1:** a nested for-loop

```python
>>> flat = []
>>> for sequence in data:
...     for item in sequence:
...         flat.append(item)
... 
>>> flat
[1, 2, 'three', 'four', 53, 54]
```

Equivalent to traditional loops would be a list comprehension with a nested
for-loop:

```python
>>> flat = [item for sequence in data for item in sequence]
>>> flat
[1, 2, 'three', 'four', 53, 54]
```

**Solution 2:** chaining the sequences using `itertools.chain`

```python
>>> from itertools import chain

>>> flat = list(chain(*data))
>>> flat
[1, 2, 'three', 'four', 53, 54]
```

Equivalent to the `list()` function call would be a list literal with the `*`
unpacking notation:

```python
>>> flat = [*chain(*data)]
>>> flat
[1, 2, 'three', 'four', 53, 54]
```

## Joining sequences of the same type

Typically, objects of the same type can be concatenated by simply summing them
together. In the following scenario, we have all our data stored in lists:

```python
numbers = [1, 2]
strings = ['three', 'four']
data = [numbers, strings]
```

Again, we intend to join `numbers` and `strings` into a single flat list.

**Solution 1:** summing the lists with the `+` operator

```python
>>> numbers + strings
[1, 2, 'three', 'four']
```

**Solution 2:** summing the lists using `functools.reduce` and
`operator.add`

```python
>>> from functools import reduce
>>> from operator import add

>>> flat = reduce(add, data)
>>> flat
[1, 2, 'three', 'four']
```

**Solution 3:** summing the lists using `sum`

```python
>>> flat = sum(data, [])
>>> flat
[1, 2, 'three', 'four']
```

Note: the [official documentation][docs_sum] suggests to use
[itertools.chain][docs_chain] rather than `sum` to concatenate iterables.

In addition, we can of course take any of the approaches demonstrated in the
previous section. For example, the list comprehension:

```python
>>> flat = [item for a_list in data for item in a_list]
>>> flat
[1, 2, 'three', 'four']
```

Or the starry list literal:

```python
>>> [*numbers, *strings]
[1, 2, 'three', 'four']
```

[docs_sum]: https://docs.python.org/3/library/functions.html#sum
[docs_chain]: https://docs.python.org/3/library/itertools.html#itertools.chain
