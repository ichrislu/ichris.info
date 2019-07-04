---
title: "group_concat和substring_index"
date: "2017-06-21 14:09:00"
lastMod: "2017-06-21 14:09:00"
tags: ["mysql", "group_concat", "substring_index"]
---

最近遇到一个较复杂的sql，研究了很久，目前仍未完美解决，其中学习到了group_concat和substring_index两个聚合函数神器，记录如下：
CONCAT函数用于将多个字符串连接成一个字符串。

CONCAT_WS 代表 CONCAT With Separator ，是CONCAT()的特殊形式。第一个参数是其它参数的分隔符。分隔符的位置放在要连接的两个字符串之间。分隔符可以是一个字符串，也可以是其它参数。如果分隔符为 NULL，则结果为 NULL。函数会忽略任何分隔符参数后的 NULL 值。但是CONCAT_WS()不会忽略任何空字符串。 (然而会忽略所有的 NULL）。

GROUP_CONCAT函数返回一个字符串结果，该结果由分组中的值连接组合而成。
GROUP_CONCAT([DISTINCT] expr [,expr ...]
[ORDER BY {unsigned_integer | col_name | formula} [ASC | DESC] [,col ...]]
[SEPARATOR str_val])
在 MySQL 中，你可以得到表达式结合体的连结值。通过使用 DISTINCT 可以排除重复值。如果希望对结果中的值进行排序，可以使用 ORDER BY 子句。
SEPARATOR 是一个字符串值，它被用于插入到结果值中。缺省为一个逗号 (",")，可以通过指定 SEPARATOR "" 完全地移除这个分隔符。
实际中什么时候需要用到这个函数？
假如需要查询的结果是这样：左边显示组名，右边想显示该组别下的所有成员信息。用这个函数，就可以省去很多事情了。

坑1：MySQL限制了group_concat返回的最大长度，默认好像是1024，超过将会被截断。
可以通过变量 group_concat_max_len 设置一个最大的长度。在运行时执行的句法如下： SET [SESSION | GLOBAL] group_concat_max_len = unsigned_integer;
如果最大长度被设置，结果值被剪切到这个最大长度。如果分组的字符过长，可以对系统参数进行设置：
SET GLOBAL group_concat_max_len=102400;  
SET SESSION group_concat_max_len=102400; 
坑2：用了GROUP_CONCAT函数，SELECT语句中的LIMIT语句起不了任何作用。

substring_index(str,delim,count)
      str:要处理的字符串
      delim:分隔符
      count:计数
如果count是正数，那么就是从左往右数，第N个分隔符的左边的全部内容
相反，如果是负数，那么就是从右边开始数，第N个分隔符右边的所有内容