---
layout: post
title: "sqlalchemy 自动创建表"
date: 2013-03-05 17:15
comments: true
categories: 
---

### 声明1个映射关系，然后如果没有表，那就自动创建

```python
#!/usr/bin/env python
# coding: utf-8


import sqlalchemy
from sqlalchemy import Column, Integer, String, DateTime, TIMESTAMP, DECIMAL
from sqlalchemy.sql.expression import text
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine
Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    fullname = Column(String(100))
    password = Column(String(32), server_default=text("123"))
    fid = Column(Integer, nullable=False, server_default=text('1'))
    d = Column(DECIMAL(2,1), nullable=False, server_default=text('0.0'))
    creation = Column(DateTime, nullable=False)
    # updated = Column(TIMESTAMP, nullable=False, server_default=text('current_timestamp on update current_timestamp'))


if __name__ == "__main__":
    engine = create_engine('sqlite:///test.db', echo=True)
    Base.metadata.create_all(engine)
```

<!-- more -->

### 自动生成语句执行

```
2013-03-08 10:05:42,805 INFO sqlalchemy.engine.base.Engine
CREATE TABLE users (
    id INTEGER NOT NULL,
    name VARCHAR(50),
    fullname VARCHAR(100),
    password VARCHAR(32) DEFAULT 123,
    fid INTEGER DEFAULT 1 NOT NULL,
    d DECIMAL(2, 1) DEFAULT 0.0 NOT NULL,
    creation DATETIME NOT NULL,
    PRIMARY KEY (id)
)

2013-03-08 10:05:42,805 INFO sqlalchemy.engine.base.Engine ()
2013-03-08 10:05:42,808 INFO sqlalchemy.engine.base.Engine COMMIT
```
