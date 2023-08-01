---
layout: post
title:  "Notes on Pattern Matching of Dictionaries"
tags: [python, data structures, pattern matching]
comments: false
---

Two peculiarities worth knowing about pattern matching of Python dicts.

## Empty dict pattern vs. empty list pattern

The following code uses an empty dictionary as the pattern for matching:

```python
def match_empty_dict(obj):
    match obj:
        case {}:
            return 'match'
    return 'no match'
```

```pycon
>>> match_empty_dict({})
'match'
>>> match_empty_dict({0: 1})
'match'
```

Both the empty and the non-empty dict successfully matched against the
`{}` pattern.

Let's compare this to a similar example that uses an empty list as the
pattern for matching:

```python
def match_empty_list(obj):
    match obj:
        case []:
            return 'match'
    return 'no match'
```

```pycon
>>> match_empty_list([])
'match'
>>> match_empty_list([0])
'no match'
```

The non-empty list did not match against the `[]` pattern.

Explanation: even though the snippets above might appear equivalent,
matching of mappings and matching of sequences are two completely
different things in Python.

## Dict literal pattern vs. dict constructor pattern

The following code uses a dictionary literal as the pattern for matching:

```python
def match_dict_literal(obj):
    match obj:
        case {'x': x}:
            return f'match {x=}'
    return 'no match'
```

```pycon
>>> match_dict_literal({'x': 1})
'match x=1'
>>> match_dict_literal({'y': 2})
'no match'
```

The `{'x': 1}` dictionary matched the `{'x': x}` pattern as expected.

Let's compare this to a similar example that uses a dictionary
constructor as the pattern for matching:

```python
def match_dict_constructor(obj):
    match obj:
        case dict(x=x):
            return f'match {x=}'
    return 'no match'
```

```pycon
>>> match_dict_constructor({'x': 1})
'no match'
>>> match_dict_constructor({'y': 2})
'no match'
```

None of the dictionaries matched the `dict(x=x)` pattern.

Explanation: even though the two code snippets above look equivalent,
they are in fact quite different. The latter syntax of the pattern
`dict(x=x)` does not take the dictionary keys into account at all, and
instead checks the object's attributes. The example dicts did not match
because none of them had an attribute named `x`. This is true for all
other classes whose objects are matched against a `cls(attribute=target)`
pattern.
