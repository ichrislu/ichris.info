---
title: "BETWEEN AND的范围"
date: "2017-01-03"
lastMod: "2017-01-03"
categories: ["it"]
tags: ["mysql", "between and"]
---

BETWEEN AND的范围是大于等于“取值1”，同时小于等于“取值2”。
例1：
```sql
SELECT * FROM employee WHERE age BETWEEN 18 AND 24;
```
age字段的取值是大于等于18，并且小于等于24。

例2：
```sql
SELECT * FROM `t_user` WHERE `f_user_register_time` BETWEEN 20170101 AND 20170102;
```
f_user_register_time字段取值为小于等于2017年1月1日0点起，至2017年1月2日0点止（**注：因为时间默认为0，所以并不是包含1月2日当天全天，仅包含1月2日0点的记录**）