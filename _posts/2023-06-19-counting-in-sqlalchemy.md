---
layout: post
title:  "Counting in SQLAlchemy"
tags: [python, sql]
comments: false
---

Basic ways to count rows of a SQL table in SQLAlchemy using the standard
`count` function.

Our example table contains three rows:

```python
with session:
    for row in session.execute(
            select(Language.id, Language.name)
    ):
        print(row)
```

```pycon
(1, 'Python')
(2, 'Go')
(3, 'Ruby')
```

Counting rows using `count('*')` and `select_from`:

```pycon
>>> query = select(func.count('*')).select_from(Language)
>>> print(query)
SELECT count(:count_2) AS count_1 
FROM language

>>> with session:
...     print(session.scalar(query))
... 
3
```

Counting rows using `count()` and `select_from`:

```pycon
>>> query = select(func.count()).select_from(Language)
>>> print(query)
SELECT count(*) AS count_1 
FROM language

>>> with session:
...     print(session.scalar(query))
... 
3
```

The latter approach seems to be preferable, as it renders the expected
`SELECT count(*)` SQL.

Alternatively, we can explicitly count the table's `id` column which serves
as the unique primary key. This allows us to skip the `select_from` method:

```pycon
>>> query = select(func.count(Language.id))
>>> print(query)
SELECT count(language.id) AS count_1 
FROM language

>>> with session:
...     print(session.scalar(query))
... 
3
```

Note: simple `count(Language)` does not work: 

```pycon
>>> query = select(func.count(Language))
Traceback (most recent call last):
...
sqlalchemy.exc.ArgumentError: SQL expression element expected, got <class 'models.Language'>.
```
