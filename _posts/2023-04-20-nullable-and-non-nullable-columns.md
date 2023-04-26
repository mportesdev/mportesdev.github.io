---
layout: post
title:  "Nullable and non-nullable columns in SQLAlchemy 2"
tags: [python, sql]
comments: false
---

Two basic ways to declare the nullable property of a table column in SQLAlchemy 2.0 [ORM].

```python
from sqlalchemy import Integer
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column


class Base(DeclarativeBase):
    ...


class Table(Base):
    __tablename__ = "table"

    id: Mapped[int] = mapped_column(primary_key=True)

    col_1: Mapped[int | None]
    col_2: Mapped[int]
    col_3 = mapped_column(Integer)
    col_4 = mapped_column(Integer, nullable=False)
```

The columns `col_1` and `col_2` are declared using only the [Mapped] type annotation.
The columns `col_3` and `col_4` are declared using the [mapped_column] function.

`col_1` and `col_3` are nullable columns:

```pycon
>>> Table.col_1.nullable
True
>>> Table.col_3.nullable
True
```

`col_2` and `col_4` are non-nullable columns:

```pycon
>>> Table.col_2.nullable
False
>>> Table.col_4.nullable
False
```

[ORM]: https://docs.sqlalchemy.org/en/20/orm/quickstart.html
[Mapped]: https://docs.sqlalchemy.org/en/20/orm/internals.html#sqlalchemy.orm.Mapped
[mapped_column]: https://docs.sqlalchemy.org/en/20/orm/mapping_api.html#sqlalchemy.orm.mapped_column
