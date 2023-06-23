---
layout: post
title:  "Notes on Pattern Matching of Dictionaries"
tags: [python, data structures, pattern matching]
comments: false
---

Two peculiarities worth knowing about pattern matching of Python dicts.

## Empty dict vs. empty list as a pattern

The following code uses an empty dictionary as a pattern for matching:

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
>>> match_empty_dict({1: 2})
'match'
```

Both the empty and the non-empty dict successfully matched against the
`{}` pattern.

Let's compare this to a seemingly equivalent example that uses an empty
list as a pattern for matching:

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
>>> match_empty_list([1, 2])
'no match'
```

The non-empty list did not match against the `[]` pattern.

## Dict literal vs. dict constructor as a pattern

The following code uses a dictionary literal as a pattern for matching:

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

Let's compare this to a seemingly equivalent example that uses a dictionary
constructor as a pattern for matching:

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
