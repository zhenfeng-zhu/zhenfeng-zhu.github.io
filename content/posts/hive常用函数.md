---
date: "2020-04-16T17:59:03+08:00"
title: hive常用函数
---

# json 字符串处理

- get_json_object
- lateral_view
- explode
- substr
- json_tuple

## get_json_object

get_json_object(string json_string, string path)

解析 json 字符串 json_string，返回 path 指定的内容。如果输入的 json 字符串是无效的，那么返回 null。

path 就是 '\$.字段名'。

如果该字段的 value 也是 json，就可以一直点下去。

如果该字段的 value 是数组，就可以用 '\$.字段名[0]'，类似这样下标的形式去访问。

## explode

explode(array)

经常和 lateral view 一起使用，将数组中的元素拆分成多行显示。

## substr

substr(string A, int start, int len)

返回字符串 A 从 start 位置开始，长度为 len 的字符串

## json_tuple

json_tuple(string json_string, col1, col2, ...)

经常和 lateral view 一起使用，同时解析多个 json 字符串中的多个字段。

# parse_url, regexp_replace, regexp_extract

## parse_url

parse_url(string urlString, string partToExtract, string keyToExtract)

返回 url 中的指定部分，如 host，path，query 等等。

partToExtract 是个枚举值：HOST, PATH, QUERY, REF, PROTOCOL, AUTHORITY, FILE, and USERINFO。

## regex_replace

regex_extract(string a, string b, string c)

将字符串 a 中符合正在表达式 b 的部分替换为 c

# json_to_struct

json_to_struct(json, 'array 或者 map 等')

# union_map

union_map(map(k, v))

# lateral view

# from_unix_time

from_unix_time(unix 时间戳, 'yyyyMMddHH')

# row_number

row_number() over (partition by 字段 a order by 计算项 b desc ) rank

hive 中的分组和组内排序

- rank 是排序的别名
- partition by： 类似于 hive 的建表，分区的意思
- order by： 排序，默认是升序，加 desc 降序

这个意思就是按字段 a 分区，对计算项 b 进行降序排列

这个是经常用到计算分区中的排序问题。

# coalesce

非空查找函数

coalesce(v1, v2, v3, ...)

返回参数中的第一个非空值，如果所有值都是 NULL，返回 NULL
