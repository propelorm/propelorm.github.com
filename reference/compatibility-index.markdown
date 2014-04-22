---
layout: documentation
title: Compatibility Index
---

# Compatibility Index

Some features are not available through all PDO supported databases. You can see here a list
of primary limitations.

At the moment we list only MySQL, SQLite and Postgres since only those are unit tested and fully supported.


## Foreign Keys

|  Functionality   |  MySQL | SQLite | Postgres
|------------------|--------|--------|---------
| General          |    √   |  √(1)  |    √ 
| `name` attribute |    √   |  X(2)  |    √ 

1) As of version 3.6.19, SQLite supports foreign key constraints.

2) SQLite does not support named foreignKeys.


## Other

|  Functionality                        |  MySQL | SQLite | Postgres
|---------------------------------------|--------|--------|---------
| Multiple autoincrement                |    √   |  X(1)  |    √ 
| Composite PK with one autoIncrement   |    √   |  √(2)  |    √ 
| `index-column` size attribute         |    √   |  X(3)  |    X(3)

1) SQLite does not support multiple columns with autoIncrement set to true.

2) SQLite does not support a composite PK where one of these is a autoIncrement. We support it anyways because
we're setting primaryKey=false and create a unique constraint instead, which is at the end basically the same.

3) Only MySQL supports a concrete size at a `index-column`.
