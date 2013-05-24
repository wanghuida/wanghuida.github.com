---
layout: post
title: "sqlalchemy 定义模型"
date: 2013-05-24 17:04
comments: true
sharing: true
footer: true
categories: [Python]
---

1. 下面是一个基本的模型，对应db的一张表
2. nullable 代表是否可以为空
3. server_default 代表默认值
4. 常用类型：Integer, String, DateTime, TIMESTAMP, DECIMAL


```python
#!/usr/bin/env python
# coding: utf-8

from sqlalchemy.sql.expression import text
from sqlalchemy import Column, Integer, String, DateTime, TIMESTAMP, DECIMAL
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()

class Member(Base):
    __tablename__ = 'tbl_member'

    id = Column(Integer, primary_key=True)
    ext_id = Column(Integer, nullable=False, server_default=text(str(0)))
    username = Column(String(64), nullable=False)
    my_desc = Column(Text)

    createtime = Column(DateTime, nullable=False)
    modifytime = Column(TIMESTAMP, nullable=False, server_default=text('CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP'))


```
