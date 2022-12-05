---
title: "joda-time使用"
date: "2017-09-05 20:41:00"
lastMod: "2017-09-06 12:18"
categories: ["it"]
tags: ["java", "joda"]
---

### 特性
- 不可变性【Immutability】：joda-time对象具有不可变性（只能在构造的时候指定，并没有set方法），所以改线安全（也有线程安全的类MutableDateTime，操作方法类同）
- 瞬间性【Instant】，暂时不明白
- 局部性【Partial】，暂时不明白
- 年表【Chronology】，暂时不明白
- 时区【Time zone】时区是值一个相对于英国格林威治的地理位置，用于计算时间。要了解事件发生的精确时间，还必须知道发生此事件的位置。任何严格的时间计算都必须涉及时区（或相对于 GMT），除非在同一个时区内发生了相对时间计算（即时这样时区也很重要，如果事件对于位于另一个时区的各方存在利益关系的话）DateTimeZone 是 Joda 库用于封装位置概念的类。许多日期和时间计算都可以在不涉及时区的情况下完成，但是仍然需要了解 DateTimeZone 如何影响 Joda 的操作。默认时间，即从运行代码的机器的系统时钟检索到的时间，在大部分情况下被使用。

### 使用


### 格式化字符
```
 Symbol  Meaning                      Presentation  Examples
 ------  -------                      ------------  -------
 G       era                          text          AD
 C       century of era (>=0)         number        20
 Y       year of era (>=0)            year          1996

 x       weekyear                     year          1996
 w       week of weekyear             number        27
 e       day of week                  number        2
 E       day of week                  text          Tuesday; Tue

 y       year                         year          1996
 D       day of year                  number        189
 M       month of year                month         July; Jul; 07
 d       day of month                 number        10

 a       halfday of day               text          PM
 K       hour of halfday (0~11)       number        0
 h       clockhour of halfday (1~12)  number        12

 H       hour of day (0~23)           number        0
 k       clockhour of day (1~24)      number        24
 m       minute of hour               number        30
 s       second of minute             number        55
 S       fraction of second           millis        978

 z       time zone                    text          Pacific Standard Time; PST
 Z       time zone offset/id          zone          -0800; -08:00; America/Los_Angeles

 '       escape for text              delimiter
 ''      single quote                 literal       '
```

### 字段说明
```
Extended      Basic       Fields
2005-03-25    20050325    year/monthOfYear/dayOfMonth
2005-03       2005-03     year/monthOfYear
2005--25      2005--25    year/dayOfMonth *
2005          2005        year
--03-25       --0325      monthOfYear/dayOfMonth
--03          --03        monthOfYear
---03         ---03       dayOfMonth
2005-084      2005084     year/dayOfYear
-084          -084        dayOfYear
2005-W12-5    2005W125    weekyear/weekOfWeekyear/dayOfWeek
2005-W-5      2005W-5     weekyear/dayOfWeek *
2005-W12      2005W12     weekyear/weekOfWeekyear
-W12-5        -W125       weekOfWeekyear/dayOfWeek
-W12          -W12        weekOfWeekyear
-W-5          -W-5        dayOfWeek
10:20:30.040  102030.040  hour/minute/second/milli
10:20:30      102030      hour/minute/second
10:20         1020        hour/minute
10            10          hour
-20:30.040    -2030.040   minute/second/milli
-20:30        -2030       minute/second
-20           -20         minute
--30.040      --30.040    second/milli
--30          --30        second
---.040       ---.040     milli *
10-30.040     10-30.040   hour/second/milli *
10:20-.040    1020-.040   hour/minute/milli *
10-30         10-30       hour/second *
10--.040      10--.040    hour/milli *
-20-.040      -20-.040    minute/milli *
```

### Quote
https://www.ibm.com/developerworks/cn/java/j-jodatime.html