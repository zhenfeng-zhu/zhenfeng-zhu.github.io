---
date: "2020-04-16T17:34:30+08:00"
title: mysql的学习
---

# SQL条件语句

## IF

if(exp1, exp2, exp3)

exp1是条件，条件为true的话，是exp2，否则是exp3

## case when

```sql

case 列名
    when 条件 then 结果
    else 其他结果
    end 别名
```

## IFNULL

IFNULL(exp1, exp2)

在exp1的值不为null的情况下，返回exp1，如果exp1位null，返回exp2的值。