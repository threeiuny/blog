---
title: 新的日期和时间API
date: 2023-09-03 18:05:38
tags: [java8, date, time, 时间]
---

## 新的日期与时间

在Java 8之前，Java的日期和时间API一直是开发者诟病的一个方面。旧的时间API由`java.util.Date`和`java.util.Calendar`类组成，这些类使用起来比较复杂，需要对`Calendar`类的内部结构有一定的了解才能使用。此外，旧的时间API不支持时区，因此无法表示来自不同时区的日期和时间。旧的时间API存在的具体缺点如下：

* **可变性**：`java.util.Date`是可变的，这意味着一旦创建了一个`Date`对象，就可以随意更改其值，导致潜在的线程安全问题。这种可变性使得日期和时间操作不可靠。

* **精度问题**：`java.util.Date`精确到毫秒级别，不支持纳秒级别的精度。在一些应用中，需要更高的精度来表示时间戳。

* **时区问题**：`java.util.Date`没有提供良好的时区支持，Date和Calendar类都是以本地时间（Local Time）的形式表示日期和时间，而不是以时区（Time Zone）的形式表示，导致了一些时区转换和计算上的困难。处理时区的代码通常变得复杂且容易出错。

* **不可读性**：旧的API中的方法名和类名不够直观，难以理解和记忆。这导致了代码的可读性较差，开发人员需要查阅文档来理解如何正确使用这些API。

* **缺少新功能**：旧的API没有提供现代日期和时间处理所需的许多功能，如计算日期之间的差异、格式化和解析日期时间字符串等。

* **不可扩展性**：旧的API很难扩展以支持不同的日历系统，如伊斯兰日历或希伯来日历。

Java 8引入了一个强大的新时间API（`java.time`），为日期和时间处理带来了显著的改进。它提供了更多的功能，更好的可读性，更好的线程安全性和更强大的时区支持。

* `java.time`：`java.time`包是Java 8引入的日期和时间处理API的核心。它提供了一组新的类和方法，用于处理日期、时间和时区。这个包的设计旨在使日期和时间操作更加直观、易于使用，并且具有更高的可读性。新API的类和方法更加丰富，可以满足各种日期和时间操作的需求。

* 不可变性：新时间API中的大多数类都是不可变的。这意味着一旦创建了一个日期或时间对象，它的值就不能被更改。不可变性是新API的关键设计原则之一，它确保了对象的线程安全性和可靠性。这意味着你可以自信地在多个线程之间共享日期和时间对象，而无需担心并发问题。

* 时区：时区是一个重要的概念，它在全球范围内管理时间的偏移量和夏令时规则。Java 8的新API引入了`ZoneId`类，用于表示时区。这允许你在处理日期和时间时考虑不同的时区。例如，你可以使用`ZoneId`将一个`LocalDateTime`对象转换为带有时区信息的`ZonedDateTime`对象，以便更准确地表示特定地点的时间。

<!-- more -->

### java.time API

Java 8引入了一组强大的日期和时间类，包括`LocalDate`、`LocalTime`、`Instant`、`Duration`和`Period`，用于处理不同类型的日期和时间信息。

#### LocalDate

`LocalDate`用于表示日期，不包含时间信息。它是不可变的，可以用于执行各种日期操作，如日期的创建、比较、计算等。

##### 日期的创建

* now()/now(ZoneId zone)/now(Clock clock)：静态工厂方法，根据系统时钟创建当前日期。`now()`方法默认使用系统默认时区创建当前日期，`now(ZoneId zone)`方法使用指定的时区创建当前日期，`now(Clock clock)`方法使用指定的时钟创建当前日期。

* of(int year, int month, int dayOfMonth)/of(int year, Month month, int dayOfMonth)：静态工厂方法，根据输入的年、月、日创建日期。`of(int year, int month, int dayOfMonth)`方法使用`int`类型的月份，`of(int year, Month month, int dayOfMonth)`方法使用`Month`枚举类型的月份。

* ofYearDay(int year, int dayOfYear)：静态工厂方法，根据输入的年份和一年中的天数创建日期。

* ofInstant(Instant instant, ZoneId zone)：静态工厂方法，根据指定的时区从`Instant`对象创建日期。

* ofEpochDay(long epochDay)：静态工厂方法，根据从1970-01-01开始的天数创建日期。

* from(TemporalAccessor temporal)：静态工厂方法，根据`TemporalAccessor`对象创建日期。

* parse(CharSequence text)/parse(CharSequence text, DateTimeFormatter formatter)：静态工厂方法，根据输入的文本创建日期。`parse(CharSequence text)`方法使用默认的格式化器创建日期，`parse(CharSequence text, DateTimeFormatter formatter)`方法使用指定的格式化器创建日期。

```java
LocalDate date1 = LocalDate.now(); // 获取当前日期
LocalDate date2 = LocalDate.now(ZoneId.of("Asia/Shanghai")); // 获取指定时区的当前日期
LocalDate date3 = LocalDate.now(Clock.systemDefaultZone()); // 获取指定时钟的当前日期
LocalDate date4 = LocalDate.of(2023, 9, 3); // 创建指定日期
LocalDate date5 = LocalDate.of(2023, Month.SEPTEMBER, 3); // 创建指定日期
LocalDate date6 = LocalDate.ofYearDay(2023, 246); // 创建指定日期
LocalDate date7 = LocalDate.ofInstant(Instant.now(), ZoneId.systemDefault()); // 创建指定时区的当前日期
LocalDate date8 = LocalDate.ofEpochDay(18628); // 创建指定日期
LocalDate date9 = LocalDate.from(date8); // 从日期创建日期
LocalDate date10 = LocalDate.parse("2023-09-03"); // 解析日期
LocalDate date11 = LocalDate.parse("2023-09-03", DateTimeFormatter.ofPattern("yyyy-MM-dd")); // 解析日期
```

##### 日期的获取

* get(TemporalField field)：获取指定字段的值。

* getLong(TemporalField field)：获取指定字段的长整型值。

* getChronology()：获取日历系统。

* getEra()：获取纪元。

* getYear()：获取年份。

* getMonth()/getMonthValue：获取月份。`getMonth()`方法返回`Month`枚举类型的月份，`getMonthValue()`方法返回`int`类型的月份。

* getDayOfMonth()：获取月份中的天数。

* getDayOfYear()：获取年份中的天数。

* getDayOfWeek()：获取星期几。

```java
int year = date1.get(ChronoField.YEAR); // 获取年份
long year1 = date1.getLong(ChronoField.YEAR); // 获取年份
int year2 = date1.getYear(); // 获取年份
Month month = date1.getMonth(); // 获取月份
int monthValue = date1.getMonthValue(); // 获取月份
int dayOfMonth = date1.getDayOfMonth(); // 获取月份中的天数
int dayOfYear = date1.getDayOfYear(); // 获取年份中的天数
DayOfWeek dayOfWeek = date1.getDayOfWeek(); // 获取星期几
```

##### 调整日期

* with(TemporalAdjuster adjuster)/with(TemporalField field, long newValue)：根据指定的调整器或字段调整日期，可使用TemporalAdjusters工具类。

* withYear(int year)/withMonth(int month)/withDayOfMonth(int dayOfMonth)/withDayOfYear(int dayOfYear)：根据指定的年、月、日调整日期。

* plus(TemporalAmount amountToAdd)/plus(long amountToAdd, TemporalUnit unit)：根据指定的时间量或单位增加日期。

* plusYears(long years)/plusMonths(long months)/plusWeeks(long weeks)/plusDays(long days)：根据指定的年、月、周、日增加日期。

* minus(TemporalAmount amountToSubtract)/minus(long amountToSubtract, TemporalUnit unit)：根据指定的时间量或单位减少日期。

* minusYears(long years)/minusMonths(long months)/minusWeeks(long weeks)/minusDays(long days)：根据指定的年、月、周、日减少日期。

* adjustInto(Temporal temporal)：调整日期。

```java
LocalDate date1 = LocalDate.now();
LocalDate date19 = date1.with(TemporalAdjusters.firstDayOfMonth()); // 设置为当月的第一天
LocalDate date2 = date1.withYear(2023); // 设置年份
LocalDate date3 = date1.withMonth(9); // 设置月份
LocalDate date4 = date1.withDayOfMonth(3); // 设置月份中的天数
LocalDate date5 = date1.withDayOfYear(246); // 设置年份中的天数
LocalDate date14 = date1.plus(Period.ofDays(1)); // 增加天数
LocalDate date15 = date1.plus(1, ChronoUnit.DAYS); // 增加天数
LocalDate date6 = date1.plusYears(1); // 增加年份
LocalDate date7 = date1.plusMonths(1); // 增加月份
LocalDate date8 = date1.plusWeeks(1); // 增加周
LocalDate date9 = date1.plusDays(1); // 增加天数
LocalDate date16 = date1.minus(Period.ofDays(1)); // 减少天数
LocalDate date17 = date1.minus(1, ChronoUnit.DAYS); // 减少天数
LocalDate date10 = date1.minusYears(1); // 减少年份
LocalDate date11 = date1.minusMonths(1); // 减少月份
LocalDate date12 = date1.minusWeeks(1); // 减少周
LocalDate date13 = date1.minusDays(1); // 减少天数
LocalDate date18 = date1.adjustInto(LocalDate.now()); // 调整日期
```

##### 比较日期

* compareTo(ChronoLocalDate other)：比较日期。

* isAfter(ChronoLocalDate other)/isBefore(ChronoLocalDate other)/isEqual(ChronoLocalDate other)：检查日期是否在指定日期之后/之前/相等。

```java
LocalDate date1 = LocalDate.now();
LocalDate date2 = LocalDate.of(2023, 9, 3);
int compare = date1.compareTo(date2); // 比较日期
boolean isAfter = date1.isAfter(date2); // 检查日期是否在指定日期之后
boolean isBefore = date1.isBefore(date2); // 检查日期是否在指定日期之前
boolean isEqual = date1.isEqual(date2); // 检查日期是否相等
```

##### 计算日期之间的差异

* until(Temporal endExclusive)/until(Temporal endExclusive, TemporalUnit unit)：计算日期之间的差异。

* datesUntil(Temporal endExclusive)/datesUntil(Temporal endExclusive, Period step)：计算日期之间的差异。

```java
LocalDate date1 = LocalDate.now();
LocalDate date2 = LocalDate.of(2023, 9, 3);
long days = date1.until(date2, ChronoUnit.DAYS); // 计算日期之间的差异
date1.datesUntil(date2).forEach(System.out::println); // 计算日期之间的差异
date1.datesUntil(date2, Period.ofDays(1)).forEach(System.out::println); // 计算日期之间的差异
```

##### 日期组合成LocalDateTime

* atTime(LocalTime time)/atTime(int hour, int minute)/atTime(int hour, int minute, int second)/atTime(int hour, int minute, int second, int nanoOfSecond)/atTime(OffsetTime time)将日期与时间组合成`LocalDateTime`对象。

* atStartOfDay()/atStartOfDay(ZoneId zone)：将日期与当天的开始时间组合成`LocalDateTime`对象。

```java
LocalDate date1 = LocalDate.now();
LocalDateTime dateTime1 = date1.atTime(LocalTime.now()); // 将日期与时间组合成LocalDateTime对象
LocalDateTime dateTime2 = date1.atTime(14, 30); // 将日期与时间组合成LocalDateTime对象
LocalDateTime dateTime3 = date1.atTime(14, 30, 0); // 将日期与时间组合成LocalDateTime对象
LocalDateTime dateTime4 = date1.atTime(14, 30, 0, 0); // 将日期与时间组合成LocalDateTime对象
LocalDateTime dateTime5 = date1.atTime(OffsetTime.now()); // 将日期与时间组合成LocalDateTime对象
LocalDateTime dateTime6 = date1.atStartOfDay(); // 将日期与当天的开始时间组合成LocalDateTime对象
LocalDateTime dateTime7 = date1.atStartOfDay(ZoneId.of("Asia/Shanghai")); // 将日期与当天的开始时间组合成LocalDateTime对象
```

##### 格式化日期

* format(DateTimeFormatter formatter)：格式化日期。

```java
LocalDate date1 = LocalDate.now();
String str1 = date1.format(DateTimeFormatter.BASIC_ISO_DATE); // 格式化日期
```

##### 日期其它

* query(TemporalQuery<R> query)：查询日期，编写自定义的查询实现，以从日期时间对象中检索特定的信息或执行特定的操作，可使用 `TemporalQueries` 工具类。

* toEpochDay()：获取从1970-01-01开始的天数。

* toEpochSecond(LocalTime time, ZoneOffset offset)：获取从1970-01-01开始的秒数。

* isSupported(TemporalField field)/isSupported(TemporalUnit unit)：检查指定的字段或单位是否支持。

* range(TemporalField field)：获取指定字段的有效值范围。

* isLeapYear()：检查是否为闰年。

* lengthOfMonth()：获取月份的天数。

* lengthOfYear()：获取年份的天数。

```java
LocalDate date1 = LocalDate.now();
LocalTime time = date.query(TemporalQueries.localTime()); // 输出null
boolean isLeapYear = date1.isLeapYear(); // 检查是否为闰年
int lengthOfMonth = date1.lengthOfMonth(); // 获取月份的天数
int lengthOfYear = date1.lengthOfYear(); // 获取年份的天数
long epochDay = date1.toEpochDay(); // 获取从1970-01-01开始的天数
long epochSecond = date1.toEpochSecond(LocalTime.now(), ZoneOffset.ofHours(8)); // 获取从1970-01-01开始的秒数
boolean isSupported1 = date1.isSupported(ChronoField.YEAR); // 检查是否支持年份
boolean isSupported2 = date1.isSupported(ChronoUnit.YEARS); // 检查是否支持年份
ValueRange range1 = date1.range(ChronoField.YEAR); // 获取年份的有效值范围
range1.getMinimum(); // 获取年份的最小值
```

#### LocalTime

`LocalTime`用于表示时间，不包含日期信息。它也是不可变的，可用于执行各种时间操作，如时间的创建、比较、计算等。

##### 时间的创建

* now()/now(ZoneId zone)/now(Clock clock)：静态工厂方法，根据系统时钟创建当前时间。`now()`方法默认使用系统默认时区创建当前时间，`now(ZoneId zone)`方法使用指定的时区创建当前时间，`now(Clock clock)`方法使用指定的时钟创建当前时间。

* of(int hour, int minute)/of(int hour, int minute, int second)/of(int hour, int minute, int second, int nanoOfSecond)：静态工厂方法，根据输入的时、分、秒、纳秒创建时间。

* ofInstant(Instant instant, ZoneId zone)：静态工厂方法，根据指定的时区从`Instant`对象创建时间。

* ofSecondOfDay(long secondOfDay)：静态工厂方法，根据从00:00开始的秒数创建时间。

* ofNanoOfDay(long nanoOfDay)：静态工厂方法，根据从00:00开始的纳秒数创建时间。

* from(TemporalAccessor temporal)：静态工厂方法，根据`TemporalAccessor`对象创建时间。

* parse(CharSequence text)/parse(CharSequence text, DateTimeFormatter formatter)：静态工厂方法，根据输入的文本创建时间。`parse(CharSequence text)`方法使用默认的格式化器创建时间，`parse(CharSequence text, DateTimeFormatter formatter)`方法使用指定的格式化器创建时间。

```java
LocalTime time1 = LocalTime.now(); // 获取当前时间
LocalTime time2 = LocalTime.now(ZoneId.of("Asia/Shanghai")); // 获取指定时区的当前时间
LocalTime time3 = LocalTime.now(Clock.systemDefaultZone()); // 获取指定时钟的当前时间
LocalTime time4 = LocalTime.of(14, 30); // 创建指定时间
LocalTime time5 = LocalTime.of(14, 30, 0); // 创建指定时间
LocalTime time6 = LocalTime.of(14, 30, 0, 0); // 创建指定时间
LocalTime time7 = LocalTime.ofInstant(Instant.now(), ZoneId.systemDefault()); // 创建指定时区的当前时间
LocalTime time8 = LocalTime.ofSecondOfDay(52200); // 创建指定时间
LocalTime time9 = LocalTime.ofNanoOfDay(52200000000000L); // 创建指定时间
LocalTime time10 = LocalTime.from(time9); // 从时间创建时间
LocalTime time11 = LocalTime.parse("14:30"); // 解析时间
LocalTime time12 = LocalTime.parse("14:30", DateTimeFormatter.ofPattern("HH:mm")); // 解析时间
```

##### 时间的获取

* get(TemporalField field)：获取指定字段的值。

* getLong(TemporalField field)：获取指定字段的长整型值。

* getHour()/getMinute()/getSecond()/getNano()：获取时、分、秒、纳秒。

```java
LocalTime time1 = LocalTime.now();
int hour = time1.get(ChronoField.HOUR_OF_DAY); // 获取小时
long hour1 = time1.getLong(ChronoField.HOUR_OF_DAY); // 获取小时
int hour2 = time1.getHour(); // 获取小时
int minute = time1.getMinute(); // 获取分钟
int second = time1.getSecond(); // 获取秒数
int nano = time1.getNano(); // 获取纳秒数
```

##### 调整时间

* with(TemporalAdjuster adjuster)/with(TemporalField field, long newValue)：根据指定的调整器或字段调整时间，可使用TemporalAdjusters工具类。

* withHour(int hour)/withMinute(int minute)/withSecond(int second)/withNano(int nanoOfSecond)：根据指定的时、分、秒、纳秒调整时间。

* truncatedTo(TemporalUnit unit)：根据指定的单位截断时间。

* plus(TemporalAmount amountToAdd)/plus(long amountToAdd, TemporalUnit unit)：根据指定的时间量或单位增加时间。

* plusHours(long hours)/plusMinutes(long minutes)/plusSeconds(long seconds)/plusNanos(long nanos)：根据指定的时、分、秒、纳秒增加时间。

* minus(TemporalAmount amountToSubtract)/minus(long amountToSubtract, TemporalUnit unit)：根据指定的时间量或单位减少时间。

* minusHours(long hours)/minusMinutes(long minutes)/minusSeconds(long seconds)/minusNanos(long nanos)：根据指定的时、分、秒、纳秒减少时间。

* adjustInto(Temporal temporal)：调整时间。

```java
LocalTime time1 = LocalTime.now();
LocalTime time20 = time.with(TemporalAdjusters.firstHourOfDay()); // 设置为当天的第一小时
LocalTime time2 = time1.withHour(14); // 设置小时
LocalTime time3 = time1.withMinute(30); // 设置分钟
LocalTime time4 = time1.withSecond(0); // 设置秒数
LocalTime time5 = time1.withNano(0); // 设置纳秒数
LocalTime time6 = time1.truncatedTo(ChronoUnit.HOURS); // 截断时间
LocalTime time7 = time1.plus(Duration.ofHours(1)); // 增加1小时
LocalTime time8 = time1.plus(1, ChronoUnit.HOURS); // 增加小时
LocalTime time9 = time1.plusHours(1); // 增加小时
LocalTime time10 = time1.plusMinutes(1); // 增加分钟
LocalTime time11 = time1.plusSeconds(1); // 增加秒数
LocalTime time12 = time1.plusNanos(1); // 增加纳秒数
LocalTime time13 = time1.minus(Duration.ofHours(1)); // 减少1小时
LocalTime time14 = time1.minus(1, ChronoUnit.HOURS); // 减少小时
LocalTime time15 = time1.minusHours(1); // 减少小时
LocalTime time16 = time1.minusMinutes(1); // 减少分钟
LocalTime time17 = time1.minusSeconds(1); // 减少秒数
LocalTime time18 = time1.minusNanos(1); // 减少纳秒数
LocalTime time19 = time1.adjustInto(LocalTime.now()); // 调整时间
```

##### 比较时间

* compareTo(LocalTime other)：比较时间。

* isAfter(LocalTime other)/isBefore(LocalTime other)/equals(Object obj)：检查时间是否在指定时间之后/之前/相等。

```java
LocalTime time1 = LocalTime.now();
LocalTime time2 = LocalTime.of(14, 30);
int compare = time1.compareTo(time2); // 比较时间
boolean isAfter = time1.isAfter(time2); // 检查时间是否在指定时间之后
boolean isBefore = time1.isBefore(time2); // 检查时间是否在指定时间之前
boolean isEqual = time1.equals(time2); // 检查时间是否相等
```

##### 计算时间的差异

* until(Temporal endExclusive, TemporalUnit unit)：计算时间之间的差异。

```java
LocalTime time1 = LocalTime.now();
LocalTime time2 = LocalTime.of(14, 30);
long hours = time1.until(time2, ChronoUnit.HOURS); // 计算时间之间的差异
```

##### 时间组合成LocalDateTime

* atDate(LocalDate date)：将时间与日期组合成`LocalDateTime`对象。

* atOffset(ZoneOffset offset)：将时间与偏移量组合成`OffsetTime`对象。

```java
LocalTime time1 = LocalTime.now();
LocalDateTime dateTime1 = time1.atDate(LocalDate.now()); // 将时间与日期组合成LocalDateTime对象
OffsetTime offsetTime1 = time1.atOffset(ZoneOffset.ofHours(8)); // 将时间与偏移量组合成OffsetTime对象
```

##### 格式化时间

* format(DateTimeFormatter formatter)：格式化时间。

```java
LocalTime time1 = LocalTime.now();
String str1 = time1.format(DateTimeFormatter.ISO_LOCAL_TIME); // 格式化时间
```

##### 时间其它

* query(TemporalQuery<R> query)：查询时间，编写自定义的查询实现，以从日期时间对象中检索特定的信息或执行特定的操作，可使用 `TemporalQueries` 工具类。

* isSupported(TemporalField field)/isSupported(TemporalUnit unit)：检查指定的字段或单位是否支持。

* range(TemporalField field)：获取指定字段的有效值范围。

* toSecondOfDay()：获取从00:00开始的秒数。

* toNanoOfDay()：获取从00:00开始的纳秒数。

* toEpochSecond(LocalDate date, ZoneOffset offset)：获取从1970-01-01开始的秒数。

```java
LocalTime time1 = LocalTime.now();
LocalTime time = time1.query(TemporalQueries.localTime()); // 输出当前时间
boolean isSupported1 = time1.isSupported(ChronoField.HOUR_OF_DAY); // 检查是否支持小时
boolean isSupported2 = time1.isSupported(ChronoUnit.HOURS); // 检查是否支持小时
ValueRange range1 = time1.range(ChronoField.HOUR_OF_DAY); // 获取小时的有效值范围
range1.getMinimum(); // 获取小时的最小值
long secondOfDay = time1.toSecondOfDay(); // 获取从00:00开始的秒数
long nanoOfDay = time1.toNanoOfDay(); // 获取从00:00开始的纳秒数
long epochSecond = time1.toEpochSecond(LocalDate.now(), ZoneOffset.ofHours(8)); // 获取从1970-01-01开始的秒数
```

#### LocalDateTime

`LocalDateTime`用于表示日期和时间，不包含时区信息。它是不可变的，可用于执行各种日期和时间操作，如日期和时间的创建、比较、计算等。

##### 日期和时间的创建

* now()/now(ZoneId zone)/now(Clock clock)：静态工厂方法，根据系统时钟创建当前日期和时间。`now()`方法默认使用系统默认时区创建当前日期和时间，`now(ZoneId zone)`方法使用指定的时区创建当前日期和时间，`now(Clock clock)`方法使用指定的时钟创建当前日期和时间。

* of(int year, int month, int dayOfMonth, int hour, int minute)/of(int year, int month, int dayOfMonth, int hour, int minute, int second)/of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond)：静态工厂方法，根据输入的年、月、日、时、分、秒、纳秒创建日期和时间。

* of(int year, Month month, int dayOfMonth, int hour, int minute)/of(int year, Month month, int dayOfMonth, int hour, int minute, int second)/of(int year, Month month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond)：静态工厂方法，根据输入的年、月、日、时、分、秒、纳秒创建日期和时间。

* of(LocalDate date, LocalTime time)：静态工厂方法，根据输入的日期和时间创建日期和时间。

* ofInstant(Instant instant, ZoneId zone)：静态工厂方法，根据指定的时区从`Instant`对象创建日期和时间。

* ofEpochSecond(long epochSecond, int nanoOfSecond, ZoneOffset offset)：静态工厂方法，根据从1970-01-01开始的秒数、纳秒数和偏移量创建日期和时间。

* from(TemporalAccessor temporal)：静态工厂方法，根据`TemporalAccessor`对象创建日期和时间。

* parse(CharSequence text)/parse(CharSequence text, DateTimeFormatter formatter)：静态工厂方法，根据输入的文本创建日期和时间。`parse(CharSequence text)`方法使用默认的格式化器创建日期和时间，`parse(CharSequence text, DateTimeFormatter formatter)`方法使用指定的格式化器创建日期和时间。

```java
LocalDateTime dateTime1 = LocalDateTime.now(); // 获取当前日期和时间
LocalDateTime dateTime2 = LocalDateTime.now(ZoneId.of("Asia/Shanghai")); // 获取指定时区的当前日期和时间
LocalDateTime dateTime3 = LocalDateTime.now(Clock.systemDefaultZone()); // 获取指定时钟的当前日期和时间
LocalDateTime dateTime4 = LocalDateTime.of(2023, 9, 3, 14, 30); // 创建指定日期和时间
LocalDateTime dateTime5 = LocalDateTime.of(2023, 9, 3, 14, 30, 0); // 创建指定日期和时间
LocalDateTime dateTime6 = LocalDateTime.of(2023, 9, 3, 14, 30, 0, 0); // 创建指定日期和时间
LocalDateTime dateTime7 = LocalDateTime.of(2023, Month.SEPTEMBER, 3, 14, 30); // 创建指定日期和时间
LocalDateTime dateTime8 = LocalDateTime.of(2023, Month.SEPTEMBER, 3, 14, 30, 0); // 创建指定日期和时间
LocalDateTime dateTime9 = LocalDateTime.of(2023, Month.SEPTEMBER, 3, 14, 30, 0, 0); // 创建指定日期和时间
LocalDateTime dateTime10 = LocalDateTime.of(LocalDate.now(), LocalTime.now()); // 创建指定日期和时间
LocalDateTime dateTime11 = LocalDateTime.ofInstant(Instant.now(), ZoneId.systemDefault()); // 创建指定时区的当前日期和时间
LocalDateTime dateTime12 = LocalDateTime.ofEpochSecond(18628, 0, ZoneOffset.ofHours(8)); // 创建指定日期和时间
LocalDateTime dateTime13 = LocalDateTime.from(dateTime12); // 从日期和时间创建日期和时间
LocalDateTime dateTime14 = LocalDateTime.parse("2023-09-03T14:30"); // 解析日期和时间
LocalDateTime dateTime15 = LocalDateTime.parse("2023-09-03 14:30", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm")); // 解析日期和时间
```

##### 日期和时间的获取

* get(TemporalField field)：获取指定字段的值。

* getLong(TemporalField field)：获取指定字段的长整型值。

* getYear()/getMonth()/getDayOfMonth()/getDayOfYear()/getDayOfWeek()/getMonthValue()：获取年、月、日、年份中的天数、星期几、月份。`getYear()`方法返回`int`类型的年份，`getMonth()`方法返回`Month`枚举类型的月份，`getDayOfMonth()`方法返回`int`类型的月份中的天数，`getDayOfYear()`方法返回`int`类型的年份中的天数，`getDayOfWeek()`方法返回`DayOfWeek`枚举类型的星期几，`getMonthValue()`方法返回`int`类型的月份。

* getHour()/getMinute()/getSecond()/getNano()：获取时、分、秒、纳秒。

##### 调整日期和时间

* with(TemporalAdjuster adjuster)/with(TemporalField field, long newValue)：根据指定的调整器或字段调整日期和时间，可使用TemporalAdjusters工具类。

* withYear(int year)/withMonth(int month)/withDayOfMonth(int dayOfMonth)/withDayOfYear(int dayOfYear)：根据指定的年、月、日、年份中的天数调整日期和时间。

* withHour(int hour)/withMinute(int minute)/withSecond(int second)/withNano(int nanoOfSecond)：根据指定的时、分、秒、纳秒调整日期和时间。

* truncatedTo(TemporalUnit unit)：根据指定的单位截断日期和时间。

* plus(TemporalAmount amountToAdd)/plus(long amountToAdd, TemporalUnit unit)：根据指定的时间量或单位增加日期和时间。

* plusYears(long years)/plusMonths(long months)/plusWeeks(long weeks)/plusDays(long days)：根据指定的年、月、周、日增加日期和时间。

* plusHours(long hours)/plusMinutes(long minutes)/plusSeconds(long seconds)/plusNanos(long nanos)：根据指定的时、分、秒、纳秒增加日期和时间。

* minus(TemporalAmount amountToSubtract)/minus(long amountToSubtract, TemporalUnit unit)：根据指定的时间量或单位减少日期和时间。

* minusYears(long years)/minusMonths(long months)/minusWeeks(long weeks)/minusDays(long days)：根据指定的年、月、周、日减少日期和时间。

* minusHours(long hours)/minusMinutes(long minutes)/minusSeconds(long seconds)/minusNanos(long nanos)：根据指定的时、分、秒、纳秒减少日期和时间。

* adjustInto(Temporal temporal)：调整日期和时间。

```java
LocalDateTime dateTime1 = LocalDateTime.now();
dateTime.with(TemporalAdjusters.firstDayOfMonth()); // 设置为当月的第一天
dateTime1.with(ChronoField.YEAR, 2023); // 设置年份
LocalDateTime dateTime2 = dateTime1.withYear(2023); // 设置年份
LocalDateTime dateTime3 = dateTime1.withMonth(9); // 设置月份
LocalDateTime dateTime4 = dateTime1.withDayOfMonth(3); // 设置月份中的天数
LocalDateTime dateTime5 = dateTime1.withDayOfYear(246); // 设置年份中的天数
LocalDateTime dateTime6 = dateTime1.withHour(14); // 设置小时
LocalDateTime dateTime7 = dateTime1.withMinute(30); // 设置分钟
LocalDateTime dateTime8 = dateTime1.withSecond(0); // 设置秒数
LocalDateTime dateTime9 = dateTime1.withNano(0); // 设置纳秒数
LocalDateTime dateTime10 = dateTime1.truncatedTo(ChronoUnit.HOURS); // 截断日期和时间
LocalDateTime dateTime11 = dateTime1.plus(Duration.ofHours(1)); // 增加1小时
LocalDateTime dateTime12 = dateTime1.plus(1, ChronoUnit.HOURS); // 增加小时
LocalDateTime dateTime13 = dateTime1.plusHours(1); // 增加小时
LocalDateTime dateTime14 = dateTime1.plusMinutes(1); // 增加分钟
LocalDateTime dateTime15 = dateTime1.plusSeconds(1); // 增加秒数
LocalDateTime dateTime16 = dateTime1.plusNanos(1); // 增加纳秒数
LocalDateTime dateTime17 = dateTime1.minus(Duration.ofHours(1)); // 减少1小时
LocalDateTime dateTime18 = dateTime1.minus(1, ChronoUnit.HOURS); // 减少小时
LocalDateTime dateTime19 = dateTime1.minusHours(1); // 减少小时
LocalDateTime dateTime20 = dateTime1.minusMinutes(1); // 减少分钟
LocalDateTime dateTime21 = dateTime1.minusSeconds(1); // 减少秒数
LocalDateTime dateTime22 = dateTime1.minusNanos(1); // 减少纳秒数
LocalDateTime dateTime23 = dateTime1.adjustInto(LocalDateTime.now()); // 调整日期和时间
```

##### 比较日期和时间

* compareTo(ChronoLocalDateTime<?> other)：比较日期和时间。

* isAfter(ChronoLocalDateTime<?> other)/isBefore(ChronoLocalDateTime<?> other)/isEqual(ChronoLocalDateTime<?> other)：检查日期和时间是否在指定日期和时间之后/之前/相等。

```java
LocalDateTime dateTime1 = LocalDateTime.now();
LocalDateTime dateTime2 = LocalDateTime.of(2023, 9, 3, 14, 30);
int compare = dateTime1.compareTo(dateTime2); // 比较日期和时间
boolean isAfter = dateTime1.isAfter(dateTime2); // 检查日期和时间是否在指定日期和时间之后
boolean isBefore = dateTime1.isBefore(dateTime2); // 检查日期和时间是否在指定日期和时间之前
boolean isEqual = dateTime1.isEqual(dateTime2); // 检查日期和时间是否相等
```

##### 计算日期和时间之间的差异

* until(Temporal endExclusive, TemporalUnit unit)：计算日期和时间之间的差异。

```java
LocalDateTime dateTime1 = LocalDateTime.now();
LocalDateTime dateTime2 = LocalDateTime.of(2023, 9, 3, 14, 30);
long hours = dateTime1.until(dateTime2, ChronoUnit.HOURS); // 计算日期和时间之间的差异
```

##### 日期和时间组合成LocalDate、LocalTime、OffsetDateTime、ZonedDateTime

* toLocalDate()：将日期和时间组合成`LocalDate`对象。

* toLocalTime()：将日期和时间组合成`LocalTime`对象。

* atOffset(ZoneOffset offset)：将日期和时间与偏移量组合成`OffsetDateTime`对象。

* atZone(ZoneId zone)：将日期和时间与时区组合成`ZonedDateTime`对象。

```java
LocalDateTime dateTime1 = LocalDateTime.now();
LocalDate date = dateTime1.toLocalDate(); // 将日期和时间组合成LocalDate对象
LocalTime time = dateTime1.toLocalTime(); // 将日期和时间组合成LocalTime对象
OffsetDateTime offsetDateTime = dateTime1.atOffset(ZoneOffset.ofHours(8)); // 将日期和时间与偏移量组合成OffsetDateTime对象
ZonedDateTime zonedDateTime = dateTime1.atZone(ZoneId.of("Asia/Shanghai")); // 将日期和时间与时区组合成ZonedDateTime对象
```

##### 格式化日期和时间

* format(DateTimeFormatter formatter)：格式化日期和时间。

```java
LocalDateTime dateTime1 = LocalDateTime.now();
String str1 = dateTime1.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME); // 格式化日期和时间
```

##### 日期和时间其它

* query(TemporalQuery<R> query)：查询日期和时间，编写自定义的查询实现，以从日期时间对象中检索特定的信息或执行特定的操作，可使用 `TemporalQueries` 工具类。

* isSupported(TemporalField field)/isSupported(TemporalUnit unit)：检查指定的字段或单位是否支持。

* range(TemporalField field)：获取指定字段的有效值范围。

* toEpochSecond(ZoneOffset offset)：获取从1970-01-01开始的秒数。

* toInstant(ZoneOffset offset)：获取从1970-01-01开始的秒数和纳秒数。

```java
LocalDateTime dateTime1 = LocalDateTime.now();
LocalDateTime dateTime = dateTime1.query(TemporalQueries.localDateTime()); // 输出当前日期和时间
boolean isSupported1 = dateTime1.isSupported(ChronoField.YEAR); // 检查是否支持年份
boolean isSupported2 = dateTime1.isSupported(ChronoUnit.YEARS); // 检查是否支持年份
ValueRange range1 = dateTime1.range(ChronoField.YEAR); // 获取年份的有效值范围
range1.getMinimum(); // 获取年份的最小值
long epochSecond = dateTime1.toEpochSecond(ZoneOffset.ofHours(8)); // 获取从1970-01-01开始的秒数
Instant instant = dateTime1.toInstant(ZoneOffset.ofHours(8)); // 获取从1970-01-01开始的秒数和纳秒数
```

#### Instant

`Instant`用于表示时间戳，精确到纳秒级别。它通常用于记录某一瞬间的时间，不受时区的影响。

##### 时间戳的创建

* now()/now(Clock clock)：静态工厂方法，根据系统时钟创建当前时间戳。`now()`方法默认使用系统默认时区创建当前时间戳，`now(Clock clock)`方法使用指定的时钟创建当前时间戳。

* ofEpochSecond(long epochSecond)/ofEpochSecond(long epochSecond, long nanoAdjustment)：静态工厂方法，根据从1970-01-01开始的秒数创建时间戳。`ofEpochSecond(long epochSecond)`方法使用0纳秒创建时间戳，`ofEpochSecond(long epochSecond, long nanoAdjustment)`方法使用指定的纳秒数创建时间戳。

* ofEpochMilli(long epochMilli)：静态工厂方法，根据从1970-01-01开始的毫秒数创建时间戳。

* from(TemporalAccessor temporal)：静态工厂方法，根据`TemporalAccessor`对象创建时间戳。

* parse(CharSequence text)：静态工厂方法，根据输入的文本创建时间戳。

```java
Instant instant1 = Instant.now(); // 获取当前时间戳
Instant instant2 = Instant.now(Clock.systemDefaultZone()); // 获取指定时钟的当前时间戳
Instant instant3 = Instant.ofEpochSecond(18628); // 创建指定时间戳
Instant instant4 = Instant.ofEpochSecond(18628, 99); // 创建指定时间戳
Instant instant5 = Instant.ofEpochMilli(18628000); // 创建指定时间戳
Instant instant6 = Instant.from(instant5); // 从时间戳创建时间戳
Instant instant7 = Instant.parse("2023-09-03T06:30:00Z"); // 解析时间戳
```

##### 时间戳的获取

* get(TemporalField field)：获取指定字段的值，只支持获取ChronoField.NANO_OF_SECOND(纳秒数)、ChronField.MICRO_OF_SECOND(微秒数)、ChronoField.MILLI_OF_SECOND(毫秒数)。

* getLong(TemporalField field)：获取指定字段的长整型值，只支持获取ChronoField.NANO_OF_SECOND(纳秒数)、ChronField.MICRO_OF_SECOND(微秒数)、ChronoField.MILLI_OF_SECOND(毫秒数)、ChronoField.INSTANT_SECONDS(从1970-01-01开始的秒数)。

* getEpochSecond()：获取从1970-01-01开始的秒数。

* getNano()：获取纳秒数，即秒数之后的余数，它的取值范围是0到999,999,999。如果需要获取完整的纳秒数，可以使用getEpochSecond()方法获取秒数，然后将纳秒数加上秒数乘以10^9。

```java
Instant instant1 = Instant.now();
int epochSecond = instant1.get(ChronoField.NANO_OF_SECOND); // 获取纳秒数
long epochSecond1 = instant1.getLong(ChronoField.INSTANT_SECONDS); // 获取从1970-01-01开始的秒数
long epochSecond2 = instant1.getEpochSecond(); // 获取从1970-01-01开始的秒数
long nano = instant1.getNano(); // 获取纳秒数
```

##### 调整时间戳

* with(TemporalAdjuster adjuster)/with(TemporalField field, long newValue)：根据指定的调整器或字段调整时间戳，可使用TemporalAdjusters工具类。

* truncatedTo(TemporalUnit unit)：根据指定的单位截断时间戳。

* plus(TemporalAmount amountToAdd)/plus(long amountToAdd, TemporalUnit unit)：根据指定的时间量或单位增加时间戳。

* plusSeconds(long seconds)/plusMillis(long millis)/plusNanos(long nanos)：根据指定的秒数、毫秒数、纳秒数增加时间戳。

* minus(TemporalAmount amountToSubtract)/minus(long amountToSubtract, TemporalUnit unit)：根据指定的时间量或单位减少时间戳。

* minusSeconds(long seconds)/minusMillis(long millis)/minusNanos(long nanos)：根据指定的秒数、毫秒数、纳秒数减少时间戳。

* adjustInto(Temporal temporal)：调整时间戳。

```java
Instant instant1 = Instant.now();
instant1.with(TemporalAdjusters.firstDayOfMonth()); // 设置为当月的第一天
Instant instant2 = instant1.with(ChronoField.NANO_OF_SECOND, 0); // 设置纳秒数
Instant instant3 = instant1.truncatedTo(ChronoUnit.SECONDS); // 截断时间戳
Instant instant4 = instant1.plus(Duration.ofSeconds(1)); // 增加1秒
Instant instant5 = instant1.plus(1, ChronoUnit.SECONDS); // 增加1秒
Instant instant6 = instant1.plusSeconds(1); // 增加1秒
Instant instant7 = instant1.plusMillis(1); // 增加1毫秒
Instant instant8 = instant1.plusNanos(1); // 增加1纳秒
Instant instant9 = instant1.minus(Duration.ofSeconds(1)); // 减少1秒
Instant instant10 = instant1.minus(1, ChronoUnit.SECONDS); // 减少1秒
Instant instant11 = instant1.minusSeconds(1); // 减少1秒
Instant instant12 = instant1.minusMillis(1); // 减少1毫秒
Instant instant13 = instant1.minusNanos(1); // 减少1纳秒
Instant instant14 = instant1.adjustInto(Instant.now()); // 调整时间戳
```

##### 比较时间戳

* compareTo(Instant other)：比较时间戳。

* isAfter(Instant other)/isBefore(Instant other)/equals(Object obj)：检查时间戳是否在指定时间戳之后/之前/相等。

```java
Instant instant1 = Instant.now();
Instant instant2 = Instant.ofEpochSecond(18628);
int compare = instant1.compareTo(instant2); // 比较时间戳
boolean isAfter = instant1.isAfter(instant2); // 检查时间戳是否在指定时间戳之后
boolean isBefore = instant1.isBefore(instant2); // 检查时间戳是否在指定时间戳之前
boolean isEqual = instant1.equals(instant2); // 检查时间戳是否相等
```

##### 计算时间戳之间的差异

* until(Temporal endExclusive, TemporalUnit unit)：计算时间戳之间的差异，只支持计算ChronoUnit.NANOS(纳秒数)、ChronoUnit.MICROS(微秒数)、ChronoUnit.MILLIS(毫秒数)、ChronoUnit.SECONDS(秒数)、ChronoUnit.MINUTES(分钟数)、ChronoUnit.HOURS(小时数)、ChronoUnit.HALF_DAYS(半天数)、ChronoUnit.DAYS(天数)。

```java
Instant instant1 = Instant.now();
Instant instant2 = Instant.ofEpochSecond(18628);
long seconds = instant1.until(instant2, ChronoUnit.SECONDS); // 计算时间戳之间的差异
```

##### 时间戳组合成LocalDateTime

* atZone(ZoneId zone)：将时间戳与时区组合成`ZonedDateTime`对象。

* atOffset(ZoneOffset offset)：将时间戳与偏移量组合成`OffsetDateTime`对象。

```java
Instant instant1 = Instant.now();
ZonedDateTime zonedDateTime1 = instant1.atZone(ZoneId.of("Asia/Shanghai")); // 将时间戳与时区组合成ZonedDateTime对象
OffsetDateTime offsetDateTime1 = instant1.atOffset(ZoneOffset.ofHours(8)); // 将时间戳与偏移量组合成OffsetDateTime对象
```

##### 时间戳其它

* query(TemporalQuery<R> query)：查询时间戳，编写自定义的查询实现，以从日期时间对象中检索特定的信息或执行特定的操作，可使用 `TemporalQueries` 工具类。

* isSupported(TemporalField field)/isSupported(TemporalUnit unit)：检查指定的字段或单位是否支持。

* range(TemporalField field)：获取指定字段的有效值范围。

* toEpochMilli()：获取从1970-01-01开始的毫秒数。

```java
Instant instant1 = Instant.now();
long epochMilli = instant1.toEpochMilli(); // 获取从1970-01-01开始的毫秒数
LocalTime time = instant1.query(TemporalQueries.localTime()); // 输出null
boolean isSupported1 = instant1.isSupported(ChronoField.NANO_OF_SECOND); // 检查是否支持纳秒数
boolean isSupported2 = instant1.isSupported(ChronoUnit.NANOS); // 检查是否支持纳秒数
ValueRange range1 = instant1.range(ChronoField.NANO_OF_SECOND); // 获取纳秒数的有效值范围
range1.getMinimum(); // 获取纳秒数的最小值
```

#### Duration

`Duration`用于表示时间段，精确到纳秒级别。它可以用于计算两个时间点之间的时间差，可以表示小时、分钟、秒、毫秒、微秒、纳秒等时间单位的持续时间。

##### 时间段的创建

* of(long amount, TemporalUnit unit)：静态工厂方法，根据指定的时间量和单位创建时间段。

* ofDays(long days)/ofHours(long hours)/ofMinutes(long minutes)/ofSeconds(long seconds)/ofSeconds(long seconds, long nanoAdjustment)/ofMillis(long millis)/ofNanos(long nanos)：静态工厂方法，根据指定的天数、小时数、分钟数、秒数、毫秒数、纳秒数创建时间段。

* from(TemporalAmount amount)：静态工厂方法，根据`TemporalAmount`对象创建时间段。

* parse(CharSequence text)：静态工厂方法，根据输入的文本创建时间段。

```java
Duration duration1 = Duration.of(1, ChronoUnit.DAYS); // 创建指定时间段
Duration duration2 = Duration.ofDays(1); // 创建指定时间段
Duration duration3 = Duration.ofHours(1); // 创建指定时间段
Duration duration4 = Duration.ofMinutes(1); // 创建指定时间段
Duration duration5 = Duration.ofSeconds(1); // 创建指定时间段
Duration duration6 = Duration.ofMillis(1); // 创建指定时间段
Duration duration7 = Duration.ofNanos(1); // 创建指定时间段
Duration duration8 = Duration.from(Duration.ofDays(1)); // 从时间段创建时间段
Duration duration9 = Duration.parse("PT24H"); // 解析时间段
```

##### 时间段的获取

* get(TemporalUnit unit)：获取指定单位的值，只支持获取ChronoUnit.SECONDS(秒数)、ChronoUnit.NANOS(纳秒数)。

* getSeconds()/getNano()：获取秒数、纳秒数。

* getUnits()：获取时间段所包含的单位（秒数、纳秒数）。

```java
Duration duration1 = Duration.ofDays(1);
long days = duration1.get(ChronoUnit.SECONDS); // 获取秒数
long seconds = duration1.getSeconds(); // 获取秒数
long nano = duration1.getNano(); // 获取纳秒数
List<TemporalUnit> units = duration1.getUnits(); // 获取时间段所包含的单位
units.forEach(unit -> System.out.println(duration1.get(unit))); // 获取时间段所包含的单位
```

##### 调整时间段

* withSeconds(long seconds)/withNanos(long nanoOfSecond)：根据指定的秒数、纳秒数调整时间段。

* plus(TemporalAmount amountToAdd)/plus(long amountToAdd, TemporalUnit unit)：根据指定的时间量或单位增加时间段。

* plusDays(long days)/plusHours(long hours)/plusMinutes(long minutes)/plusSeconds(long seconds)/plusMillis(long millis)/plusNanos(long nanos)：根据指定的天数、小时数、分钟数、秒数、毫秒数、纳秒数增加时间段。

* minus(TemporalAmount amountToSubtract)/minus(long amountToSubtract, TemporalUnit unit)：根据指定的时间量或单位减少时间段。

* minusDays(long days)/minusHours(long hours)/minusMinutes(long minutes)/minusSeconds(long seconds)/minusMillis(long millis)/minusNanos(long nanos)：根据指定的天数、小时数、分钟数、秒数、毫秒数、纳秒数减少时间段。

* multipliedBy(long multiplicand)：根据指定的乘数乘以时间段。

* dividedBy(long divisor)/dividedBy(Duration divisor)：根据指定的除数除以时间段。

* negated()：将时间段取反。

* abs()：获取时间段的绝对值。

* addTo(Temporal temporal)：将时间段添加到指定的时间对象上。

* subtractFrom(Temporal temporal)：从指定的时间对象上减去时间段。

* truncatedTo(TemporalUnit unit)：根据指定的单位截断时间段。

```java
Duration duration1 = Duration.ofDays(1);
Duration duration2 = duration1.withSeconds(1); // 设置秒数
Duration duration3 = duration1.plus(Duration.ofDays(1)); // 增加1天
Duration duration4 = duration1.plus(1, ChronoUnit.DAYS); // 增加1天
Duration duration5 = duration1.plusDays(1); // 增加1天
Duration duration6 = duration1.plusHours(1); // 增加1小时
Duration duration7 = duration1.plusMinutes(1); // 增加1分钟
Duration duration8 = duration1.plusSeconds(1); // 增加1秒
Duration duration9 = duration1.plusMillis(1); // 增加1毫秒
Duration duration10 = duration1.plusNanos(1); // 增加1纳秒
Duration duration11 = duration1.minus(Duration.ofDays(1)); // 减少1天
Duration duration12 = duration1.minus(1, ChronoUnit.DAYS); // 减少1天
Duration duration13 = duration1.minusDays(1); // 减少1天
Duration duration14 = duration1.minusHours(1); // 减少1小时
Duration duration15 = duration1.minusMinutes(1); // 减少1分钟
Duration duration16 = duration1.minusSeconds(1); // 减少1秒
Duration duration17 = duration1.minusMillis(1); // 减少1毫秒
Duration duration18 = duration1.minusNanos(1); // 减少1纳秒
Duration duration19 = duration1.multipliedBy(2); // 乘以2
Duration duration20 = duration1.dividedBy(2); // 除以2
Duration duration21 = duration1.dividedBy(Duration.ofDays(1)); // 除以1天
Duration duration22 = duration1.negated(); // 取反
Duration duration23 = duration1.abs(); // 获取绝对值
Duration duration24 = duration1.truncatedTo(ChronoUnit.DAYS); // 截断时间段
```

##### 比较时间段

* compareTo(Duration other)：比较时间段。

* isNegative()/isZero()：检查时间段是否为负数/零。

* between(Temporal startInclusive, Temporal endExclusive)：计算两个时间点之间的时间段。

* equals(Object obj)：检查时间段是否相等。

```java
Duration duration1 = Duration.ofDays(1);
Duration duration2 = Duration.ofDays(2);
int compare = duration1.compareTo(duration2); // 比较时间段
boolean isNegative = duration1.isNegative(); // 检查时间段是否为负数
boolean isZero = duration1.isZero(); // 检查时间段是否为零
Duration duration3 = Duration.between(Instant.now(), Instant.now().plus(Duration.ofDays(1))); // 计算两个时间点之间的时间段
boolean isEqual = duration1.equals(duration2); // 检查时间段是否相等
```

##### 将时间段转换成其它类型

* toDays()/toHours()/toMinutes()/toSeconds()/toMillis()/toNanos()：将时间段转换成天数、小时数、分钟数、秒数、毫秒数、纳秒数。

* toDaysPart()/toHoursPart()/toMinutesPart()/toSecondsPart()/toMillisPart()/toNanosPart()：将时间段转换成天数、小时数、分钟数、秒数、毫秒数、纳秒数的部分。

```java
Duration duration1 = Duration.ofDays(1);
long days = duration1.toDays(); // 获取天数
long hours = duration1.toHours(); // 获取小时数
long minutes = duration1.toMinutes(); // 获取分钟数
long seconds = duration1.getSeconds(); // 获取秒数
long millis = duration1.toMillis(); // 获取毫秒数
long nanos = duration1.toNanos(); // 获取纳秒数
long daysPart = duration1.toDaysPart(); // 获取天数的部分
long hoursPart = duration1.toHoursPart(); // 获取小时数的部分
long minutesPart = duration1.toMinutesPart(); // 获取分钟数的部分
long secondsPart = duration1.toSecondsPart(); // 获取秒数的部分
long millisPart = duration1.toMillisPart(); // 获取毫秒数的部分
long nanosPart = duration1.toNanosPart(); // 获取纳秒数的部分
```

#### Period

`Period`用于表示日期段，以年、月、日为单位。它通常用于计算两个日期之间的日期差异，精度为天级别，可以表示年、月、日等时间单位的持续时间。

##### 日期段的创建

* of(int years, int months, int days)：静态工厂方法，根据指定的年、月、日创建日期段。

* ofYears(int years)/ofMonths(int months)/ofWeeks(int weeks)/ofDays(int days)：静态工厂方法，根据指定的年、月、周、日创建日期段。

* from(TemporalAmount amount)：静态工厂方法，根据`TemporalAmount`对象创建日期段。

* parse(CharSequence text)：静态工厂方法，根据输入的文本创建日期段。

```java
Period period1 = Period.of(1, 2, 3); // 创建指定日期段
Period period2 = Period.ofYears(1); // 创建指定日期段
Period period3 = Period.ofMonths(1); // 创建指定日期段
Period period4 = Period.ofWeeks(1); // 创建指定日期段
Period period5 = Period.ofDays(1); // 创建指定日期段
Period period6 = Period.from(Period.ofYears(1)); // 从日期段创建日期段
Period period7 = Period.parse("P1Y2M3D"); // 解析日期段
```

##### 日期段的获取

* get(TemporalUnit unit)：获取指定单位的值，只支持获取ChronoUnit.YEARS(年数)、ChronoUnit.MONTHS(月数)、ChronoUnit.DAYS(天数)。

* getYears()/getMonths()/getDays()：获取年数、月数、天数。

* getUnits()：获取日期段所包含的单位（年数、月数、天数）。

```java
Period period1 = Period.of(1, 2, 3);
long years = period1.get(ChronoUnit.YEARS); // 获取年数
long months = period1.getMonths(); // 获取月数
long days = period1.getDays(); // 获取天数
List<TemporalUnit> units = period1.getUnits(); // 获取日期段所包含的单位
units.forEach(unit -> System.out.println(period1.get(unit))); // 获取日期段所包含的单位
```

##### 调整日期段

* withYears(int years)/withMonths(int months)/withDays(int days)：根据指定的年数、月数、天数调整日期段。

* plus(TemporalAmount amountToAdd)：根据指定的时间量增加日期段。

* plusYears(long years)/plusMonths(long months)/plusDays(long days)：根据指定的年数、月数、天数增加日期段。

* minus(TemporalAmount amountToSubtract)：根据指定的时间量减少日期段。

* minusYears(long years)/minusMonths(long months)/minusDays(long days)：根据指定的年数、月数、天数减少日期段。

* multipliedBy(int scalar)：根据指定的乘数乘以日期段。

* negated()：将日期段取反。

* normalized()：将日期段标准化，将月数和天数调整到合理的范围内，例如一个日期段为1年15个月45天，那么标准化后为2年3个月15天。

* addTo(Temporal temporal)：将日期段添加到指定的时间对象上。

* subtractFrom(Temporal temporal)：从指定的时间对象上减去日期段。

```java
Period period1 = Period.of(1, 2, 3);
Period period2 = period1.withYears(1); // 设置年数
Period period3 = period1.plus(Period.ofYears(1)); // 增加1年
Period period5 = period1.plusYears(1); // 增加1年
Period period6 = period1.plusMonths(1); // 增加1月
Period period7 = period1.plusDays(1); // 增加1天
Period period8 = period1.minus(Period.ofYears(1)); // 减少1年
Period period10 = period1.minusYears(1); // 减少1年
Period period11 = period1.minusMonths(1); // 减少1月
Period period12 = period1.minusDays(1); // 减少1天
Period period13 = period1.multipliedBy(2); // 乘以2
Period period14 = period1.negated(); // 取反
Period period15 = period1.normalized(); // 标准化
LocalDate period16 = (LocalDate)periods.addTo(LocalDate.now()); // 添加到日期上
LocalDate period17 = (LocalDate)periods.subtractFrom(LocalDate.now()); // 从日期上减去
```

##### 比较日期段

* between(LocalDate startDateInclusive, LocalDate endDateExclusive)：计算两个日期之间的日期段。

* isZero()/isNegative：检查日期段是否为零/负数。

* equals(Object obj)：检查日期段是否相等。

```java
Period period1 = Period.of(1, 2, 3);
period1.isZero(); // 检查日期段是否为零
period1.isNegative(); // 检查日期段是否为负数
Period period2 = Period.between(LocalDate.now(), LocalDate.now().plusDays(1)); // 计算两个日期之间的日期段
boolean isEqual = period1.equals(period2); // 检查日期段是否相等
```

##### 日期段其它API

* getChronology()：获取日期段的日历系统的历法，默认为ISO。

* toTotalMonths()：获取日期段的总月数。

```java
Period period1 = Period.of(1, 2, 3);
Chronology chronology = period1.getChronology(); // 获取日期段的日历系统
int totalMonths = period1.toTotalMonths(); // 获取日期段的总月数
```

#### TemporalField与ChronoField

`TemporalField`用于表示日期时间对象中的字段（例如年、月、日、小时、分钟等）。TemporalField的主要作用是允许您访问和操作日期时间对象的特定字段，以获取或修改它们的值。

`ChronoField`是`TemporalField`的实现类，它是一个枚举类，它定义了一些常用的日期时间字段，如下所示：

* NANO_OF_SECOND：秒内的纳秒数，取值范围为0~999,999,999。

* NANO_OF_DAY：一天中的纳秒数，取值范围为0~86,399,999,999。

* MICRO_OF_SECOND：秒内的微秒数，取值范围为0~999,999。

* MICRO_OF_DAY：一天中的微秒数，取值范围为0~86,399,999。

* MILLI_OF_SECOND：秒内的毫秒数，取值范围为0~999。

* MILLI_OF_DAY：一天中的毫秒数，取值范围为0~86,399,999。

* SECOND_OF_MINUTE：分钟内的秒数，取值范围为0~59。

* SECOND_OF_DAY：一天中的秒数，取值范围为0~86,399。

* MINUTE_OF_HOUR：小时内的分钟数，取值范围为0~59。

* MINUTE_OF_DAY：一天中的分钟数，取值范围为0~1,439。

* HOUR_OF_DAY：一天中的小时数，取值范围为0~23。

* HOUR_OF_AMPM：上午或下午的小时数，取值范围为0~11。

* CLOCK_HOUR_OF_DAY：一天中的小时数（12小时制），取值范围为1~12。

* CLOCK_HOUR_OF_AMPM：上午或下午的小时数（12小时制），取值范围为1~12。

* AMPM_OF_DAY：上午或下午，取值范围为0（上午）或1（下午）。

* DAY_OF_WEEK：一周中的天数，取值范围为1（周一）~7（周日）。

* DAY_OF_MONTH：一个月中的天数，取值范围为1~31。

* DAY_OF_YEAR：一年中的天数，取值范围为1~365或366。

* EPOCH_DAY：从1970-01-01开始的天数，取值范围为-365,243~365,243。

* ALIGNED_WEEK_OF_MONTH：一个月中的周数，取值范围为1~5。

* ALIGNED_WEEK_OF_YEAR：一年中的周数，取值范围为1~53。

* MONTH_OF_YEAR：一年中的月数，取值范围为1~12。

* PROLEPTIC_MONTH：从公元前开始计算的月数，取值范围为-999,999,999~999,999,999。

* YEAR：年份。

* YEAR_OF_ERA：纪元年份，取值范围为1~n。

* ERA：时代，取值范围为0（公元前）或1（公元）。

* INSTANT_SECONDS：从1970-01-01开始的秒数。

* OFFSET_SECONDS：时区偏移量，取值范围为-64800~64800。

`ChronoField`定义的日期时间字段可用于Temporal对象时间的获取和调整，例如：

```java
LocalDate date1 = LocalDate.now();
LocalDate date2 = date1.with(ChronoField.YEAR, 2020); // 将年份调整到2020年
int year = date1.get(ChronoField.YEAR); // 获取年份
```

#### TemporalUnit与ChronoUnit

`TemporalUnit`是Java 8日期时间API中的一个接口，用于表示日期时间的时间单位。其允许开发人员在日期时间对象之间执行时间差计算，并以不同的时间单位表示这些差值。其常用方法如下：

* addTo(Temporal temporal, long amount)：将指定的时间量（amount）添加到给定的Temporal对象中，生成一个新的Temporal对象。

* between(Temporal temporal1Inclusive, Temporal temporal2Exclusive)：计算两个Temporal对象之间的时间量，通常用于计算两个日期时间对象之间的时间差。

`TemporalUnit`接口的实现类有`ChronoUnit`、`IsoFields`、`JulianFields`等。`ChronoUnit`是`TemporalUnit`的实现类，它是一个枚举类，它定义了一些常用的时间单位，如下所示：

* DAYS：天数。

* WEEKS：周数。

* MONTHS：月数。

* YEARS：年数。

* DECADES：十年数。

* CENTURIES：百年数。

* MILLENNIA：千年数。

* ERAS：时代数。

* FOREVER：永远。

* NANOS：纳秒数。

* MICROS：微秒数。

* MILLIS：毫秒数。

* SECONDS：秒数。

* MINUTES：分钟数。

* HOURS：小时数。

* HALF_DAYS：半天数。

ChronoUnit定义的时间单位可用于Temporal对象时间调整时使用，例如：

```java
LocalDate date1 = LocalDate.now();
LocalDate date2 = date1.with(ChronoUnit.DAYS, 1); // 将日期调整到明天
LocalDate date3 = date1.with(ChronoUnit.MONTHS, 1); // 将日期调整到下个月
LocalDate date4 = date1.plus(1, ChronoUnit.DAYS); // 将日期调整到明天
```

除此之外，ChronUnit还可以用于计算时间差和调整时间段，例如：

```java
LocalDate date1 = LocalDate.now();
LocalDate date2 = LocalDate.now().plus(1, ChronoUnit.DAYS);
long days = ChronoUnit.DAYS.between(date1, date2); // 计算两个日期之间的天数
// 调整时间
LocalDate date = LocalDate.now();
date.with(ChronoUnit.MONTHS.addTo(date, 1));
```

##### ChronoUnit与ChronoField的不同

`ChronoField`和`ChronoUnit`都是Java 8日期时间API中用于表示日期和时间的不同方面的枚举类型，它们虽然都包含了大量的枚举值，但它们的作用和用途有很大的不同。

* `ChronoField`：

  * **作用**：`ChronoField`是用于表示日期时间中的具体字段（例如年、月、日、小时、分钟等）的枚举类型。每个`ChronoField`枚举值代表一个具体的日期时间字段。`ChronoField`用于访问和操作日期时间对象的具体字段值。它通常用于获取和设置日期时间对象中的特定字段，例如，获取年份、月份、小时等具体的日期时间信息。

* `ChronoUnit`：

  * **作用**：`ChronoUnit`是用于表示日期时间之间的时间单位（例如年、月、日、小时、分钟、秒等）的枚举类型。每个`ChronoUnit`枚举值代表一个时间单位。`ChronoUnit`通常用于执行日期时间之间的时间差计算，例如，计算两个日期之间相差的天数、小时数、分钟数等。

**主要区别**：

* `ChronoField`关注日期时间对象的具体字段，它用于访问和操作日期时间中的年、月、日、小时等具体信息。

* `ChronoUnit`关注日期时间之间的时间单位，它用于执行日期时间之间的时间差计算，例如，计算两个日期之间相差的年、月、日、小时等时间单位。

#### TemporalAccessor与TemporalQuery

`java.time`包中的`TemporalAccessor`和`TemporalQuery`是Java 8日期时间API中的两个重要接口，它们用于处理日期和时间对象的访问和查询。

`TemporalAccessor`接口是一个表示可以读取日期时间信息的对象的通用接口。它提供了一组方法，允许从日期时间对象中提取各种字段的值，例如年、月、日、小时、分钟、秒等。`TemporalAccessor`的实现类包括`LocalDate`、`LocalTime`、`LocalDateTime`、`ZonedDateTime`等。

`TemporalAccessor`的主要方法包括：

* get(TemporalField field)：根据指定的`TemporalField`获取字段的值。

* getLong(TemporalField field)：以长整型形式获取字段的值。

* query(TemporalQuery<R> query)：使用指定的`TemporalQuery`执行自定义查询操作。

```java
LocalDate date1 = LocalDate.now();
int year = date1.get(ChronoField.YEAR); // 获取年份
long longYear = date1.getLong(ChronoField.YEAR); // 获取年份
```

`TemporalQuery`接口是一个函数式接口，用于执行自定义的日期时间查询操作。它定义了一个单一的抽象方法`R queryFrom(TemporalAccessor temporal)`，该方法接受一个`TemporalAccessor`对象作为参数，并返回一个泛型类型`R`的结果。

`TemporalQuery`的主要作用是根据特定的需求从日期时间对象中提取信息。开发人员可以编写自定义的查询实现，以从日期时间对象中检索特定的信息或执行特定的操作。这可以实现灵活地处理日期时间数据，根据具体需求进行查询和操作。如下所示：

```java
LocalDate date = LocalDate.now();
TemporalQuery<String> stringTemporalQuery = temporal -> "Year: " + temporal.get(ChronoField.YEAR) + ", Month: " + temporal.get(ChronoField.MONTH_OF_YEAR) + ", Day: " + temporal.get(ChronoField.DAY_OF_MONTH);
// 也可以使用如下方式来自定义查询
// TemporalQuery<String> stringTemporalQuery = temporal -> {
//     LocalDate localDate = LocalDate.from(temporal);
//     return  "Year: " + localDate.getYear() + ", Month: " + localDate.getMonthValue() + ", Day: " + localDate.getDayOfMonth();
// };
System.out.println(date.query(stringTemporalQuery));
```

#### 使用TemporalAdjuster调整日期

`TemporalAdjuster`用于调整日期，它是一个函数式接口，只有一个`adjustInto(Temporal temporal)`方法，该方法用于调整日期。TemporalAdjuster可以用于调整Temporal对象的时间，Temporal对象可以是LocalDate、LocalDateTime、LocalTime、Instant、ZonedDateTime、OffsetDateTime等。其对应的工具类是TemporalAdjusters，它提供了一些静态方法，用于创建TemporalAdjuster对象。

* dayOfWeekInMonth(int ordinal, DayOfWeek dayOfWeek)：创建一个TemporalAdjuster对象，用于将日期调整到指定月份的第几个星期几。

* firstDayOfMonth()：创建一个TemporalAdjuster对象，用于将日期调整到当月的第一天。

* firstDayOfNextMonth()：创建一个TemporalAdjuster对象，用于将日期调整到下个月的第一天。

* firstDayOfNextYear()：创建一个TemporalAdjuster对象，用于将日期调整到下一年的第一天。

* firstDayOfYear()：创建一个TemporalAdjuster对象，用于将日期调整到当年的第一天。

* firstInMonth(DayOfWeek dayOfWeek)：创建一个TemporalAdjuster对象，用于将日期调整到当月的第一个星期几。

* lastDayOfMonth()：创建一个TemporalAdjuster对象，用于将日期调整到当月的最后一天。

* lastDayOfYear()：创建一个TemporalAdjuster对象，用于将日期调整到当年的最后一天。

* lastInMonth(DayOfWeek dayOfWeek)：创建一个TemporalAdjuster对象，用于将日期调整到当月的最后一个星期几。

* next(DayOfWeek dayOfWeek)：创建一个TemporalAdjuster对象，用于将日期调整到下一个星期几。

* nextOrSame(DayOfWeek dayOfWeek)：创建一个TemporalAdjuster对象，用于将日期调整到下一个或当天的星期几。

* previous(DayOfWeek dayOfWeek)：创建一个TemporalAdjuster对象，用于将日期调整到上一个星期几。

* previousOrSame(DayOfWeek dayOfWeek)：创建一个TemporalAdjuster对象，用于将日期调整到上一个或当天的星期几。

* ofDateAdjuster(UnaryOperator<LocalDate> dateBasedAdjuster)：创建一个TemporalAdjuster对象，用于将日期调整到指定的日期。

```java
LocalDate date1 = LocalDate.now();
LocalDate date2 = date1.with(TemporalAdjusters.firstDayOfMonth()); // 将日期调整到当月的第一天
LocalDate date3 = date1.with(TemporalAdjusters.firstDayOfNextMonth()); // 将日期调整到下个月的第一天
LocalDate date4 = date1.with(TemporalAdjusters.firstDayOfNextYear()); // 将日期调整到下一年的第一天
LocalDate date5 = date1.with(TemporalAdjusters.firstDayOfYear()); // 将日期调整到当年的第一天
LocalDate date6 = date1.with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY)); // 将日期调整到当月的第一个星期一
LocalDate date7 = date1.with(TemporalAdjusters.lastDayOfMonth()); // 将日期调整到当月的最后一天
LocalDate date8 = date1.with(TemporalAdjusters.lastDayOfYear()); // 将日期调整到当年的最后一天
LocalDate date9 = date1.with(TemporalAdjusters.lastInMonth(DayOfWeek.MONDAY)); // 将日期调整到当月的最后一个星期一
LocalDate date10 = date1.with(TemporalAdjusters.next(DayOfWeek.MONDAY)); // 将日期调整到下一个星期一
LocalDate date11 = date1.with(TemporalAdjusters.nextOrSame(DayOfWeek.MONDAY)); // 将日期调整到下一个或当天的星期一
LocalDate date12 = date1.with(TemporalAdjusters.previous(DayOfWeek.MONDAY)); // 将日期调整到上一个星期一
LocalDate date13 = date1.with(TemporalAdjusters.previousOrSame(DayOfWeek.MONDAY)); // 将日期调整到上一个或当天的星期一
LocalDate date14 = date1.with(TemporalAdjusters.ofDateAdjuster(localDate -> localDate.plusDays(1))); // 将日期调整到下一天
```

#### 使用DateTimeFormatter格式化日期和时间

`DateTimeFormatter`用于格式化日期和时间，它是线程安全的，可以通过静态工厂方法创建DateTimeFormatter对象，也可以通过DateTimeFormatterBuilder类创建DateTimeFormatter对象。

DaTeTimeFormatter定义了很多常用时间的格式化枚举，如下所示：

* ISO_LOCAL_DATE：日期格式，例如：2019-01-01。

* ISO_LOCAL_TIME：时间格式，例如：10:15:30。

* ISO_LOCAL_DATE_TIME：日期时间格式，例如：2019-01-01T10:15:30。

* ISO_INSTANT：时间戳格式，例如：2019-01-01T10:15:30Z。

* ISO_OFFSET_DATE_TIME：带偏移量的日期时间格式，例如：2019-01-01T10:15:30+08:00。

* ISO_ZONED_DATE_TIME：带时区的日期时间格式，例如：2019-01-01T10:15:30+08:00[Asia/Shanghai]。

* ISO_DATE：日期格式，例如：2019-01-01+08:00。

* ISO_TIME：时间格式，例如：10:15:30+08:00。

* ISO_DATE_TIME：日期时间格式，例如：2019-01-01T10:15:30+08:00。

* ISO_ORDINAL_DATE：日期格式，例如：2019-001。

* ISO_WEEK_DATE：日期格式，例如：2019-W01-2。

* BASIC_ISO_DATE：日期格式，例如：20190101。

* RFC_1123_DATE_TIME：日期时间格式，例如：Tue, 1 Jan 2019 10:15:30 +0800。

* ISO_OFFSET_DATE：带偏移量的日期格式，例如：2019-01-01+08:00。

* ISO_OFFSET_TIME：带偏移量的时间格式，例如：10:15:30+08:00。

DateTimeFormatter定义了很多常用的格式化模式，如下所示：

* ofPattern(String pattern)/ofPattern(String pattern, Locale locale)：静态工厂方法，根据指定的格式化模式和区域创建DateTimeFormatter对象。

* ofLocalizedDate(FormatStyle dateStyle)/ofLocalizedTime(FormatStyle timeStyle)/ofLocalizedDateTime(FormatStyle dateTimeStyle)/ofLocalizedDateTime(FormatStyle dateTimeStyle, FormatStyle timeStyle)：静态工厂方法，根据指定的日期、时间、日期时间格式创建DateTimeFormatter对象。dateStyle指定了日期格式的样式:

  * SHORT：短格式，例如：2019-1-1。

  * MEDIUM：中等格式，例如：2019-01-01。

  * LONG：长格式，例如：2019年1月1日，LONG格式时，需要被格式化的时间携带时区信息（ZonedDateTIme）或者设置时区信息(DateTimeFormatter.withZone(ZoneId zone))。

  * FULL：完整格式，例如：2019年1月1日星期二， FULL格式时，需要被格式化的时间携带时区信息（ZonedDateTIme）或者设置时区信息(DateTimeFormatter.withZone(ZoneId zone))。

* parsedExcessDays()/parsedLeapSecond()：解析格式化器中包含的额外天数和闰秒信息。

* withLocale(Locale locale)/getLocale()：设置/获取区域。

* getDecimalStyle()/withDecimalStyle(DecimalStyle decimalStyle)：设置/获取十进制样式。

* getChronology()/withChronology(Chronology chrono)：设置/获取日历系统。

* getZone()/withZone(ZoneId zone)：设置/获取时区。

* withResolverStyle(ResolverStyle resolverStyle)/getResolverStyle()：设置/获取解析样式。

* getResolverFields()/withResolverFields(TemporalField... resolverFields)/withResolverFields(Set<TemporalField> resolverFields)：设置/获取解析字段。

* format(TemporalAccessor temporal)/format(TemporalAccessor temporal, Appendable appendable)：格式化日期、时间。

* parse(CharSequence text)/parse(CharSequence text, ParsePosition position)：解析日期、时间。

* parse(CharSequence text, TemporalQuery<T> query)/parseBest(CharSequence text, TemporalQuery<?>... queries)：解析日期、时间。

* parseUnresolved(CharSequence text, ParsePosition position)：解析日期、时间。

* toFormat()/toFormat(TemporalQuery<?> parseQuery)：将DateTimeFormatter对象转换成Format对象。

```java
LocalDateTime dateTime1 = LocalDateTime.now();
String text1 = dateTime1.format(DateTimeFormatter.ISO_LOCAL_DATE); // 格式化日期
DateTimeFormatter.ofPattern("yyyy-MM-dd").format(dateTime1); // 格式化日期
DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss", Locale.CHINA).format(dateTime1); // 格式化日期
DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL).format(LocalDate.now()); // 格式化日期
DateTimeFormatter.ofLocalizedTime(FormatStyle.FULL).format(LocalDateTime.now().atZone(ZoneId.systemDefault())); // 格式化时间
DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL).withZone(ZoneId.systemDefault()).format(dateTime1); // 格式化日期时间
DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL, FormatStyle.FULL).format(LocalDateTime.now().atZone(ZoneId.systemDefault())); // 格式化日期时间
DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss").withLocale(Locale.CHINA).format(dateTime1); // 格式化日期
DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss").withLocale(Locale.CHINA).withZone(ZoneId.systemDefault()).format(dateTime1); // 格式化日期
TemporalAccessor temporalAccessor = DateTimeFormatter.ofPattern("yyyy-MM-dd").parse("2023-09-12");
LocalDate.from(temporalAccessor);
```

#### 处理不同时区的日期和时间

时区是地球上的不同地区根据经度的划分，每个时区都有自己的标准时间偏移和可能的夏令时规则。时区信息包括了某个地区的标准时间和夏令时的开始与结束时间。为什么需要时区？时区的存在是为了确保在不同地理位置的计算机和设备上，时间可以正确地同步和表示。没有时区信息，相同的时间可能在不同地区被解释为不同的时刻，这可能会导致时间混淆和计算错误。在许多实际应用中，时区信息非常重要。例如，跨时区的会议安排、国际航班时间表、金融交易、协调全球时间等情况都需要精确的时区信息。在这些情况下，使用`LocalDateTime`是不够的，需要使用带有时区信息的类，如`ZonedDateTime`，它可以表示日期和时间以及相对于特定时区的信息。

##### 协调世界时（UTC）与夏令时（DST）

协调世界时（UTC）是世界上的标准时间，它是世界上所有时区的参考时间。UTC时间不会因为夏令时的改变而改变，它是一个固定的时间。UTC时间是以原子钟为基础的，它的精确度非常高，误差不超过1秒。UTC时间是以24小时制表示的，例如：下午3点是15:00，下午3点30分是15:30。

夏令时（DST，Daylight Saving Time）也称为日光节约时间，是一种为节约能源而人为规定地方时间的制度，在这一制度实行期间所采用的统一时间称为“夏令时间”。夏令时一般在天亮早的夏季人为将时间调快一小时，可以使人早起早睡，减少照明量，以充分利用光照资源，从而节约照明用电。夏令时通常在春季开始，秋季结束。具体实施时间各个国家和地区有所不同。

协调世界时是基于原子时秒长为基础的一种时间计量系统，并不随季节变化，而夏令时是人为规定的地方时间，因此在夏令时期间，同一时刻的夏令时间与UTC时间相差一个小时，这些问题会导致在处理时间相关业务时产生混淆和错误。

为了解决这些问题，Java 8日期时间API分别使用`ZoneId`和`ZoneOffset`来表示时区和时区偏移量；使用ZonedDateTime类用于表示带有时区信息的日期和时间，使用OffsetDateTime类用于表示带有时区偏移量的日期和时间。

##### ZoneId

`ZoneId`类用于表示时区的标识符，它表示地理区域内的时区，包括夏令时转换等信息。`ZoneId`解决了跨越多个地理区域的日期和时间表示问题，它允许将日期时间对象关联到特定的地理位置，以便正确处理时区转换和夏令时变更。

`ZoneId`常用方法如下：

* of(String zoneId)/of(String zoneId, Map<String, String> aliasMap)：静态工厂方法，根据指定的时区ID创建ZoneId对象。

* ofOffset(String prefix, ZoneOffset offset)：静态工厂方法，根据指定的时区ID前缀和偏移量创建ZoneId对象。

* getAvailableZoneIds()：获取所有可用的时区ID。

* systemDefault()：获取系统默认时区。

* from(TemporalAccessor temporal)：根据指定的日期时间对象获取时区。

* getDisplayName(TextStyle style, Locale locale)：获取时区的显示名称。

* getRules()：获取时区的规则。

* normalized()：获取规范的时区。

* equals(Object obj)：检查时区是否相等。

```java
ZoneId zoneId1 = ZoneId.of("Asia/Shanghai"); // 创建时区
ZoneId zoneId2 = ZoneId.of("GMT+08:00"); // 创建时区
ZoneId zoneId3 = ZoneId.systemDefault(); // 获取系统默认时区
Set<String> availableZoneIds = ZoneId.getAvailableZoneIds(); // 获取所有可用的时区ID
ZoneId zoneId4 = ZoneId.from(ZonedDateTime.now()); // 根据指定的日期时间对象获取时区
String displayName = zoneId1.getDisplayName(TextStyle.FULL, Locale.CHINA); // 获取时区的显示名称，输出中国时间
ZoneRules rules = zoneId1.getRules(); // 获取时区的规则
ZoneId zoneId5 = zoneId1.normalized(); // 获取规范的时区，输出Asia/Shanghai
boolean isEqual = zoneId1.equals(zoneId2); // 检查时区是否相等
```

##### ZoneOffset

`ZoneOffset`用于表示与协调世界时 (UTC) 的固定时间偏移，例如+02:00、-05:00等，通常用于表示不考虑夏令时的固定偏移时区，`ZoneId`提供了固定的偏移信息，不包括时区的地理区域信息，解决了处理不考虑夏令时的日期时间表示问题。

`ZoneOffset`常用方法如下：

* of(String offsetId)/ofHours(int hours)/ofHoursMinutes(int hours, int minutes)/ofHoursMinutesSeconds(int hours, int minutes, int seconds)：静态工厂方法，根据指定的时区偏移量创建ZoneOffset对象。

* from(TemporalAccessor temporal)：根据指定的日期时间对象获取时区偏移量。

* ofTotalSeconds(int totalSeconds)：根据指定的总秒数创建ZoneOffset对象。

* getTotalSeconds()：获取总秒数。

* getRules()：获取时区偏移量的规则。

* isSupported(TemporalField field)：检查是否支持指定的字段。

* range(TemporalField field)：获取指定字段的值范围。

* get(TemporalField field)/getLong(TemporalField field)：获取指定字段的值。

* query(TemporalQuery<R> query)：使用指定的`TemporalQuery`执行自定义查询操作。

* adjustInto(Temporal temporal)：将时区偏移量添加到指定的时间对象上。

* compareTo(ZoneOffset other)：比较两个时区偏移量。

* equals(Object obj)：检查时区偏移量是否相等。

```java
ZoneOffset zoneOffset1 = ZoneOffset.of("+08:00"); // 创建时区偏移量
ZoneOffset zoneOffset2 = ZoneOffset.ofHours(8); // 创建时区偏移量
ZoneOffset zoneOffset3 = ZoneOffset.ofHoursMinutes(8, 0); // 创建时区偏移量
ZoneOffset zoneOffset4 = ZoneOffset.ofHoursMinutesSeconds(8, 0, 0); // 创建时区偏移量
ZoneOffset zoneOffset5 = ZoneOffset.from(ZonedDateTime.now()); // 根据指定的日期时间对象获取时区偏移量
ZoneOffset zoneOffset6 = ZoneOffset.ofTotalSeconds(28800); // 根据指定的总秒数创建ZoneOffset对象
int totalSeconds = zoneOffset1.getTotalSeconds(); // 获取总秒数
ZoneRules rules = zoneOffset1.getRules(); // 获取时区偏移量的规则
boolean isSupported = zoneOffset1.isSupported(ChronoField.YEAR); // 检查是否支持指定的字段
ValueRange valueRange = zoneOffset1.range(ChronoField.YEAR); // 获取指定字段的值范围
long year = zoneOffset1.getLong(ChronoField.YEAR); // 获取指定字段的值
ZoneOffset zoneOffset7 = zoneOffset1.adjustInto(ZonedDateTime.now()); // 将时区偏移量添加到指定的时间对象上
int compareResult = zoneOffset1.compareTo(zoneOffset2); // 比较两个时区偏移量
boolean isEqual = zoneOffset1.equals(zoneOffset2); // 检查时区偏移量是否相等
```

##### ZonedDateTime

`ZonedDateTime`用于表示带有时区信息的日期和时间。它包括了日期、时间和时区，能够处理夏令时变更，可以精确地表示全球范围内的日期和时间。

###### 时区信息的日期和时间创建

* of(LocalDate date, LocalTime time, ZoneId zone)/of(LocalDateTime dateTime, ZoneId zone)/of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond, ZoneId zone)：静态工厂方法，根据指定的日期、时间、时区创建ZonedDateTime对象。

* ofInstant(Instant instant, ZoneId zone)/ofInstant(LocalDateTime localDateTime, ZoneOffset offset, ZoneId zone)：将`Instant`或`LocalDateTime`对象转换为`ZonedDateTime`对象。

* ofStrict(LocalDateTime localDateTime, ZoneOffset offset, ZoneId zone)：静态工厂方法，根据指定的日期时间、时区和时区偏移量创建ZonedDateTime对象。

* ofLocal(LocalDateTime localDateTime, ZoneId zone, ZoneOffset preferredOffset)：静态工厂方法，根据指定的日期时间和时区创建ZonedDateTime对象。

* now()/now(ZoneId zone)/now(Clock clock)：静态工厂方法，根据指定的时区或时钟创建ZonedDateTime对象。

* from(TemporalAccessor temporal)：根据指定的日期时间对象获取ZonedDateTime对象。

* parse(CharSequence text)/parse(CharSequence text, DateTimeFormatter formatter)：静态工厂方法，根据指定的日期时间字符串创建ZonedDateTime对象。

```java
ZonedDateTime zonedDateTime1 = ZonedDateTime.of(LocalDate.now(), LocalTime.now(), ZoneId.of("Asia/Shanghai")); // 创建ZonedDateTime对象
ZonedDateTime zonedDateTime2 = ZonedDateTime.of(LocalDateTime.now(), ZoneId.of("Asia/Shanghai")); // 创建ZonedDateTime对象
ZonedDateTime zonedDateTime3 = ZonedDateTime.of(2019, 1, 1, 10, 15, 30, 0, ZoneId.of("Asia/Shanghai")); // 创建ZonedDateTime对象
ZonedDateTime zonedDateTime4 = ZonedDateTime.ofInstant(Instant.now(), ZoneId.of("Asia/Shanghai")); // 将Instant对象转换为ZonedDateTime对象
ZonedDateTime zonedDateTime5 = ZonedDateTime.ofInstant(LocalDateTime.now(), ZoneOffset.ofHours(8), ZoneId.of("Asia/Shanghai")); // 将LocalDateTime对象转换为ZonedDateTime对象
ZonedDateTime zonedDateTime6 = ZonedDateTime.ofStrict(LocalDateTime.now(), ZoneOffset.ofHours(8), ZoneId.of("Asia/Shanghai")); // 创建ZonedDateTime对象
ZonedDateTime zonedDateTime7 = ZonedDateTime.ofLocal(LocalDateTime.now(), ZoneId.of("Asia/Shanghai"), ZoneOffset.ofHours(8)); // 创建ZonedDateTime对象
ZonedDateTime zonedDateTime8 = ZonedDateTime.now(); // 创建ZonedDateTime对象
ZonedDateTime zonedDateTime9 = ZonedDateTime.now(ZoneId.of("Asia/Shanghai")); // 创建ZonedDateTime对象
ZonedDateTime zonedDateTime10 = ZonedDateTime.now(Clock.systemDefaultZone()); // 创建ZonedDateTime对象
ZonedDateTime zonedDateTime11 = ZonedDateTime.from(ZonedDateTime.now()); // 根据指定的日期时间对象获取ZonedDateTime对象
ZonedDateTime zonedDateTime12 = ZonedDateTime.parse("2019-01-01T10:15:30+08:00[Asia/Shanghai]"); // 根据指定的日期时间字符串创建ZonedDateTime对象
ZonedDateTime zonedDateTime13 = ZonedDateTime.parse("2019-01-01T10:15:30+08:00[Asia/Shanghai]", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ssZ")); // 根据指定的日期时间字符串创建ZonedDateTime对象
```

###### 时区信息的日期和时间的获取

* get(TemporalField field)/getLong(TemporalField field)：获取指定字段的值。

* getYears()/getMonths()/getMonthValue()/getDayOfMonth()/getDayOfYear()/getDayOfWeek()/getHour()/getMinute()/getSecond()/getNano()：获取年、月、日、小时、分钟、秒、纳秒等。

```java
ZonedDateTime zonedDateTime = ZonedDateTime.now();
int year = zonedDateTime.get(ChronoField.YEAR); // 获取年份
long longYear = zonedDateTime.getLong(ChronoField.YEAR); // 获取年份
int month = zonedDateTime.get(ChronoField.MONTH_OF_YEAR); // 获取月份
int year1 = zonedDateTime.getYear(); // 获取年份
Month month1 = zonedDateTime.getMonth(); // 获取月份
int monthValue = zonedDateTime.getMonthValue(); // 获取月份
int dayOfMonth1 = zonedDateTime.getDayOfMonth(); // 获取日
int dayOfYear1 = zonedDateTime.getDayOfYear(); // 获取年中的第几天
DayOfWeek dayOfWeek1 = zonedDateTime.getDayOfWeek(); // 获取周几
int hour1 = zonedDateTime.getHour(); // 获取小时
int minute1 = zonedDateTime.getMinute(); // 获取分钟
int second1 = zonedDateTime.getSecond(); // 获取秒
int nano1 = zonedDateTime.getNano(); // 获取纳秒
```

###### 获取时区、时区的偏移量及调整时区和时区的偏移量

* getOffset()：获取时区偏移量。

* getZone()：获取时区。

* withEarlierOffsetAtOverlap()/withLaterOffsetAtOverlap()：在重叠的时候，使用较早的时区偏移量或较晚的时区偏移量。

* withZoneSameLocal(ZoneId zone)：将时区调整为指定的时区，保持本地时间不变。

* withZoneSameInstant(ZoneId zone)：将时区调整为指定的时区，保持时刻不变。

* withFixedOffsetZone()：将时区调整为固定的偏移量时区。

```java
ZonedDateTime zonedDateTime = ZonedDateTime.now();
ZoneOffset offset = zonedDateTime.getOffset(); // 获取时区偏移量
ZoneId zone = zonedDateTime.getZone(); // 获取时区
ZonedDateTime zonedDateTime1 = zonedDateTime.withEarlierOffsetAtOverlap(); // 在重叠的时候，使用较早的时区偏移量
ZonedDateTime zonedDateTime2 = zonedDateTime.withLaterOffsetAtOverlap(); // 在重叠的时候，使用较晚的时区偏移量
ZonedDateTime zonedDateTime3 = zonedDateTime.withZoneSameLocal(ZoneId.of("Asia/Shanghai")); // 将时区调整为指定的时区，保持本地时间不变
ZonedDateTime zonedDateTime4 = zonedDateTime.withZoneSameInstant(ZoneId.of("Asia/Shanghai")); // 将时区调整为指定的时区，保持时刻不变
ZonedDateTime zonedDateTime5 = zonedDateTime.withFixedOffsetZone(); // 将时区调整为固定的偏移量时区
```

###### 获取本地时间信息

* toLocalDateTime()：获取本地日期时间。

* toLocalDate()：获取本地日期。

* toLocalTime()：获取本地时间。

* toOffsetDateTime()：将ZonedDateTime对象转换为OffsetDateTime对象。

```java
ZonedDateTime zonedDateTime = ZonedDateTime.now();
LocalDateTime localDateTime = zonedDateTime.toLocalDateTime(); // 获取本地日期时间
LocalDate localDate = zonedDateTime.toLocalDate(); // 获取本地日期
LocalTime localTime = zonedDateTime.toLocalTime(); // 获取本地时间
OffsetDateTime offsetDateTime = zonedDateTime.toOffsetDateTime(); // 将ZonedDateTime对象转换为OffsetDateTime对象
```

###### 调整时区的日期和时间

* with(TemporalAdjuster adjuster)/with(TemporalField field, long newValue)：根据指定的调整器或字段调整日期时间。

* withYear()/withMonth()/withDayOfMonth()/withDayOfYear()/withHour()/withMinute()/withSecond()/withNano()：设置年、月、日、小时、分钟、秒、纳秒等。

* truncatedTo(TemporalUnit unit)：截断日期时间。

* plus(TemporalAmount amountToAdd)/plus(long amountToAdd, TemporalUnit unit)：增加日期时间。

* plusYears()/plusMonths()/plusWeeks()/plusDays()/plusHours()/plusMinutes()/plusSeconds()/plusNanos()：增加年、月、周、日、小时、分钟、秒、纳秒等。

* minus(TemporalAmount amountToSubtract)/minus(long amountToSubtract, TemporalUnit unit)：减少日期时间。

* minusYears()/minusMonths()/minusWeeks()/minusDays()/minusHours()/minusMinutes()/minusSeconds()/minusNanos()：减少年、月、周、日、小时、分钟、秒、纳秒等。

```java
ZonedDateTime zonedDateTime = ZonedDateTime.now();
ZonedDateTime zonedDateTime1 = zonedDateTime.with(TemporalAdjusters.firstDayOfMonth()); // 将日期调整到当月的第一天
ZonedDateTime zonedDateTime2 = zonedDateTime.with(ChronoField.YEAR, 2019); // 将年份调整为2019
ZonedDateTime zonedDateTime3 = zonedDateTime.withYear(2019); // 将年份调整为2019
ZonedDateTime zonedDateTime4 = zonedDateTime.withMonth(1); // 将月份调整为1
ZonedDateTime zonedDateTime5 = zonedDateTime.withDayOfMonth(1); // 将日调整为1
ZonedDateTime zonedDateTime6 = zonedDateTime.withDayOfYear(1); // 将年中的第几天调整为1
ZonedDateTime zonedDateTime7 = zonedDateTime.withHour(1); // 将小时调整为1
ZonedDateTime zonedDateTime8 = zonedDateTime.withMinute(1); // 将分钟调整为1
ZonedDateTime zonedDateTime9 = zonedDateTime.withSecond(1); // 将秒调整为1
ZonedDateTime zonedDateTime10 = zonedDateTime.withNano(1); // 将纳秒调整为1
ZonedDateTime zonedDateTime11 = zonedDateTime.truncatedTo(ChronoUnit.DAYS); // 截断日期时间
ZonedDateTime zonedDateTime12 = zonedDateTime.plus(1, ChronoUnit.DAYS); // 增加1天
ZonedDateTime zonedDateTime13 = zonedDateTime.plusYears(1); // 增加1年
ZonedDateTime zonedDateTime14 = zonedDateTime.plusMonths(1); // 增加1月
ZonedDateTime zonedDateTime15 = zonedDateTime.plusWeeks(1); // 增加1周
ZonedDateTime zonedDateTime16 = zonedDateTime.plusDays(1); // 增加1天
ZonedDateTime zonedDateTime17 = zonedDateTime.plusHours(1); // 增加1小时
ZonedDateTime zonedDateTime18 = zonedDateTime.plusMinutes(1); // 增加1分钟
ZonedDateTime zonedDateTime19 = zonedDateTime.plusSeconds(1); // 增加1秒
ZonedDateTime zonedDateTime20 = zonedDateTime.plusNanos(1); // 增加1纳秒
ZonedDateTime zonedDateTime21 = zonedDateTime.minus(1, ChronoUnit.DAYS); // 减少1天
ZonedDateTime zonedDateTime22 = zonedDateTime.minusYears(1); // 减少1年
ZonedDateTime zonedDateTime23 = zonedDateTime.minusMonths(1); // 减少1月
ZonedDateTime zonedDateTime24 = zonedDateTime.minusWeeks(1); // 减少1周
ZonedDateTime zonedDateTime25 = zonedDateTime.minusDays(1); // 减少1天
ZonedDateTime zonedDateTime26 = zonedDateTime.minusHours(1); // 减少1小时
ZonedDateTime zonedDateTime27 = zonedDateTime.minusMinutes(1); // 减少1分钟
ZonedDateTime zonedDateTime28 = zonedDateTime.minusSeconds(1); // 减少1秒
ZonedDateTime zonedDateTime29 = zonedDateTime.minusNanos(1); // 减少1纳秒
```

###### 计算两个时区的日期时间的差

* until(Temporal endExclusive, TemporalUnit unit)：计算两个日期时间之间的时间差。

```java
ZonedDateTime zonedDateTime1 = ZonedDateTime.now();
ZonedDateTime zonedDateTime2 = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
long days = zonedDateTime1.until(zonedDateTime2, ChronoUnit.DAYS); // 计算两个日期时间之间的时间差
```

###### 格式化时区的日期和时间

* format(DateTimeFormatter formatter)：格式化日期时间。

```java
ZonedDateTime zonedDateTime = ZonedDateTime.now();
String text = zonedDateTime.format(DateTimeFormatter.ISO_LOCAL_DATE); // 格式化日期
String text1 = zonedDateTime.format(DateTimeFormatter.ISO_LOCAL_TIME); // 格式化时间
```

###### 时区的日期和时间的其它

* query(TemporalQuery<R> query)：使用指定的`TemporalQuery`执行自定义查询操作。

* equals(Object obj)：检查日期时间是否相等。

* isSupported(TemporalField field)/isSupported(TemporalUnit unit)：检查是否支持指定的字段或时间单位。

* range(TemporalField field)：获取指定字段的值范围。

```java
ZonedDateTime zonedDateTime = ZonedDateTime.now();
boolean isSupported = zonedDateTime.isSupported(ChronoField.YEAR); // 检查是否支持指定的字段
boolean isSupported1 = zonedDateTime.isSupported(ChronoUnit.DAYS); // 检查是否支持指定的时间单位
ValueRange valueRange = zonedDateTime.range(ChronoField.YEAR); // 获取指定字段的值范围
boolean isEqual = zonedDateTime.equals(ZonedDateTime.now()); // 检查日期时间是否相等
```

##### OffsetDateTime

`OffsetDateTime`用于表示带有固定时间偏移的日期和时间，表示的是一个具有固定偏移的时间。

###### 创建固定时间偏移的日期和时间

* of(LocalDate date, LocalTime time, ZoneOffset offset)/of(LocalDateTime dateTime, ZoneOffset offset)/of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond, ZoneOffset offset)：静态工厂方法，根据指定的日期、时间、时区偏移量创建OffsetDateTime对象。

* ofInstant(Instant instant, ZoneId zone)：将`Instant`对象转换为`OffsetDateTime`对象。

* now()/now(ZoneId zone)/now(Clock clock)：静态工厂方法，根据指定的时区或时钟创建OffsetDateTime对象。

* from(TemporalAccessor temporal)：根据指定的日期时间对象获取OffsetDateTime对象。

* parse(CharSequence text)/parse(CharSequence text, DateTimeFormatter formatter)：静态工厂方法，根据指定的日期时间字符串创建OffsetDateTime对象。

```java
OffsetDateTime offsetDateTime1 = OffsetDateTime.of(LocalDate.now(), LocalTime.now(), ZoneOffset.ofHours(8)); // 创建OffsetDateTime对象
OffsetDateTime offsetDateTime2 = OffsetDateTime.of(LocalDateTime.now(), ZoneOffset.ofHours(8)); // 创建OffsetDateTime对象
OffsetDateTime offsetDateTime3 = OffsetDateTime.of(2019, 1, 1, 10, 15, 30, 0, ZoneOffset.ofHours(8)); // 创建OffsetDateTime对象
OffsetDateTime offsetDateTime4 = OffsetDateTime.ofInstant(Instant.now(), ZoneId.of("Asia/Shanghai")); // 将Instant对象转换为OffsetDateTime对象
OffsetDateTime offsetDateTime5 = OffsetDateTime.now(); // 创建OffsetDateTime对象
OffsetDateTime offsetDateTime6 = OffsetDateTime.now(ZoneId.of("Asia/Shanghai")); // 创建OffsetDateTime对象
OffsetDateTime offsetDateTime7 = OffsetDateTime.now(Clock.systemDefaultZone()); // 创建OffsetDateTime对象
OffsetDateTime offsetDateTime8 = OffsetDateTime.from(OffsetDateTime.now()); // 根据指定的日期时间对象获取OffsetDateTime对象
OffsetDateTime offsetDateTime9 = OffsetDateTime.parse("2019-01-01T10:15:30+08:00"); // 根据指定的日期时间字符串创建OffsetDateTime对象
OffsetDateTime offsetDateTime10 = OffsetDateTime.parse("2019-01-01T10:15:30+08:00", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ssZ")); // 根据指定的日期时间字符串创建OffsetDateTime对象
```

###### 获取固定时间偏移的日期和时间的信息

* get(TemporalField field)/getLong(TemporalField field)：获取指定字段的值。

* getYear()/getMonthValue()/getDayOfMonth()/getDayOfYear()/getDayOfWeek()/getHour()/getMinute()/getSecond()/getNano()：获取年、月、日、小时、分钟、秒、纳秒等。

```java
OffsetDateTime offsetDateTime = OffsetDateTime.now();
int year = offsetDateTime.get(ChronoField.YEAR); // 获取年份
long longYear = offsetDateTime.getLong(ChronoField.YEAR); // 获取年份
int year1 = offsetDateTime.getYear(); // 获取年份
int month = offsetDateTime.getMonthValue(); // 获取月份
int dayOfMonth1 = offsetDateTime.getDayOfMonth(); // 获取日
int dayOfYear1 = offsetDateTime.getDayOfYear(); // 获取年中的第几天
DayOfWeek dayOfWeek1 = offsetDateTime.getDayOfWeek(); // 获取周几
int hour1 = offsetDateTime.getHour(); // 获取小时
int minute1 = offsetDateTime.getMinute(); // 获取分钟
int second1 = offsetDateTime.getSecond(); // 获取秒
int nano1 = offsetDateTime.getNano(); // 获取纳秒
```

###### 获取固定时区的偏移量

* getOffset()：获取时区偏移量。

* withOffsetSameLocal(ZoneOffset offset)：将时区偏移量调整为指定的时区偏移量，保持本地时间不变。

* withOffsetSameInstant(ZoneOffset offset)：将时区偏移量调整为指定的时区偏移量，保持时刻不变。

* atZoneSameInstant(ZoneId zone)：将OffsetDateTime对象转换为ZonedDateTime对象。

* atZoneSameLocal(ZoneId zone)：将OffsetDateTime对象转换为ZonedDateTime对象。

```java
OffsetDateTime offsetDateTime = OffsetDateTime.now();
ZoneOffset offset = offsetDateTime.getOffset(); // 获取时区偏移量
OffsetDateTime offsetDateTime1 = offsetDateTime.withOffsetSameLocal(ZoneOffset.ofHours(8)); // 将时区偏移量调整为指定的时区偏移量，保持本地时间不变
OffsetDateTime offsetDateTime2 = offsetDateTime.withOffsetSameInstant(ZoneOffset.ofHours(8)); // 将时区偏移量调整为指定的时区偏移量，保持时刻不变
ZonedDateTime zonedDateTime = offsetDateTime.atZoneSameInstant(ZoneId.of("Asia/Shanghai")); // 将OffsetDateTime对象转换为ZonedDateTime对象
ZonedDateTime zonedDateTime1 = offsetDateTime.atZoneSameLocal(ZoneId.of("Asia/Shanghai")); // 将OffsetDateTime对象转换为ZonedDateTime对象
```

###### 固定时区获取本地时间信息

* toLocalDateTime()：获取本地日期时间。

* toLocalDate()：获取本地日期。

* toLocalTime()：获取本地时间。

* toZonedDateTime()：将OffsetDateTime对象转换为ZonedDateTime对象。

* toOffsetTime()：将OffsetDateTime对象转换为OffsetTime对象。

* toInstant()：将OffsetDateTime对象转换为Instant对象。

* toEpochSecond()：将OffsetDateTime对象转换为秒数。

```java
OffsetDateTime offsetDateTime = OffsetDateTime.now();
LocalDateTime localDateTime = offsetDateTime.toLocalDateTime(); // 获取本地日期时间
LocalDate localDate = offsetDateTime.toLocalDate(); // 获取本地日期
LocalTime localTime = offsetDateTime.toLocalTime(); // 获取本地时间
ZonedDateTime zonedDateTime = offsetDateTime.toZonedDateTime(); // 将OffsetDateTime对象转换为ZonedDateTime对象
OffsetTime offsetTime = offsetDateTime.toOffsetTime(); // 将OffsetDateTime对象转换为OffsetTime对象
Instant instant = offsetDateTime.toInstant(); // 将OffsetDateTime对象转换为Instant对象
long epochSecond = offsetDateTime.toEpochSecond(); // 将OffsetDateTime对象转换为秒数
```

###### 固定时区调整日期和时间

* with(TemporalAdjuster adjuster)/with(TemporalField field, long newValue)：根据指定的调整器或字段调整日期时间。

* withYear()/withMonth()/withDayOfMonth()/withDayOfYear()/withHour()/withMinute()/withSecond()/withNano()：设置年、月、日、小时、分钟、秒、纳秒等。

* truncatedTo(TemporalUnit unit)：截断日期时间。

* plus(TemporalAmount amountToAdd)/plus(long amountToAdd, TemporalUnit unit)：增加日期时间。

* plusYears()/plusMonths()/plusWeeks()/plusDays()/plusHours()/plusMinutes()/plusSeconds()/plusNanos()：增加年、月、周、日、小时、分钟、秒、纳秒等。

* minus(TemporalAmount amountToSubtract)/minus(long amountToSubtract, TemporalUnit unit)：减少日期时间。

* minusYears()/minusMonths()/minusWeeks()/minusDays()/minusHours()/minusMinutes()/minusSeconds()/minusNanos()：减少年、月、周、日、小时、分钟、秒、纳秒等。

* adjustInto(Temporal temporal)：将时区偏移量添加到指定的时间对象上。

```java
OffsetDateTime offsetDateTime = OffsetDateTime.now();
OffsetDateTime offsetDateTime1 = offsetDateTime.with(TemporalAdjusters.firstDayOfMonth()); // 将日期调整到当月的第一天
OffsetDateTime offsetDateTime2 = offsetDateTime.with(ChronoField.YEAR, 2019); // 将年份调整为2019
OffsetDateTime offsetDateTime3 = offsetDateTime.withYear(2019); // 将年份调整为2019
OffsetDateTime offsetDateTime4 = offsetDateTime.withMonth(1); // 将月份调整为1
OffsetDateTime offsetDateTime5 = offsetDateTime.withDayOfMonth(1); // 将日调整为1
OffsetDateTime offsetDateTime6 = offsetDateTime.withDayOfYear(1); // 将年中的第几天调整为1
OffsetDateTime offsetDateTime7 = offsetDateTime.withHour(1); // 将小时调整为1
OffsetDateTime offsetDateTime8 = offsetDateTime.withMinute(1); // 将分钟调整为1
OffsetDateTime offsetDateTime9 = offsetDateTime.withSecond(1); // 将秒调整为1
OffsetDateTime offsetDateTime10 = offsetDateTime.withNano(1); // 将纳秒调整为1
OffsetDateTime offsetDateTime11 = offsetDateTime.truncatedTo(ChronoUnit.DAYS); // 截断日期时间
OffsetDateTime offsetDateTime12 = offsetDateTime.plus(1, ChronoUnit.DAYS); // 增加1天
OffsetDateTime offsetDateTime13 = offsetDateTime.plusYears(1); // 增加1年
OffsetDateTime offsetDateTime14 = offsetDateTime.plusMonths(1); // 增加1月
OffsetDateTime offsetDateTime15 = offsetDateTime.plusWeeks(1); // 增加1周
OffsetDateTime offsetDateTime16 = offsetDateTime.plusDays(1); // 增加1天
OffsetDateTime offsetDateTime17 = offsetDateTime.plusHours(1); // 增加1小时
OffsetDateTime offsetDateTime18 = offsetDateTime.plusMinutes(1); // 增加1分钟
OffsetDateTime offsetDateTime19 = offsetDateTime.plusSeconds(1); // 增加1秒
OffsetDateTime offsetDateTime20 = offsetDateTime.plusNanos(1); // 增加1纳秒
OffsetDateTime offsetDateTime21 = offsetDateTime.minus(1, ChronoUnit.DAYS); // 减少1天
OffsetDateTime offsetDateTime22 = offsetDateTime.minusYears(1); // 减少1年
OffsetDateTime offsetDateTime23 = offsetDateTime.minusMonths(1); // 减少1月
OffsetDateTime offsetDateTime24 = offsetDateTime.minusWeeks(1); // 减少1周
OffsetDateTime offsetDateTime25 = offsetDateTime.minusDays(1); // 减少1天
OffsetDateTime offsetDateTime26 = offsetDateTime.minusHours(1); // 减少1小时
OffsetDateTime offsetDateTime27 = offsetDateTime.minusMinutes(1); // 减少1分钟
OffsetDateTime offsetDateTime28 = offsetDateTime.minusSeconds(1); // 减少1秒
OffsetDateTime offsetDateTime29 = offsetDateTime.minusNanos(1); // 减少1纳秒
OffsetDateTime offsetDateTime29 = offsetDateTime.adjustInto(ZonedDateTime.now()); // 将时区偏移量添加到指定的时间对象上
```

###### 计算两个固定时区的日期时间的差

* until(Temporal endExclusive, TemporalUnit unit)：计算两个日期时间之间的时间差。

```java
OffsetDateTime offsetDateTime1 = OffsetDateTime.now();
OffsetDateTime offsetDateTime2 = OffsetDateTime.now(ZoneOffset.ofHours(8));
long days = offsetDateTime1.until(offsetDateTime2, ChronoUnit.DAYS); // 计算两个日期时间之间的时间差
```

###### 比较两个固定时区的日期时间

* compareTo(OffsetDateTime other)：比较两个日期时间。

* isAfter(OffsetDateTime other)：检查日期时间是否在指定日期时间之后。

* isBefore(OffsetDateTime other)：检查日期时间是否在指定日期时间之前。

* isEqual(OffsetDateTime other)：检查日期时间是否相等。

```java
OffsetDateTime offsetDateTime1 = OffsetDateTime.now();
OffsetDateTime offsetDateTime2 = OffsetDateTime.now(ZoneOffset.ofHours(8));
int compareResult = offsetDateTime1.compareTo(offsetDateTime2); // 比较两个日期时间
boolean isAfter = offsetDateTime1.isAfter(offsetDateTime2); // 检查日期时间是否在指定日期时间之后
boolean isBefore = offsetDateTime1.isBefore(offsetDateTime2); // 检查日期时间是否在指定日期时间之前
boolean isEqual = offsetDateTime1.isEqual(offsetDateTime2); // 检查日期时间是否相等
```

###### 格式化固定时区的日期和时间

* format(DateTimeFormatter formatter)：格式化日期时间。

```java
OffsetDateTime offsetDateTime = OffsetDateTime.now();
String text = offsetDateTime.format(DateTimeFormatter.ISO_LOCAL_DATE); // 格式化日期
```

###### 固定时区的日期和时间的其它

* query(TemporalQuery<R> query)：使用指定的`TemporalQuery`执行自定义查询操作。

* isSupported(TemporalField field)/isSupported(TemporalUnit unit)：检查是否支持指定的字段或时间单位。

* range(TemporalField field)：获取指定字段的值范围。

* timeLineOrder()：获取时间线顺序。

```java
OffsetDateTime offsetDateTime = OffsetDateTime.now();
boolean isSupported = offsetDateTime.isSupported(ChronoField.YEAR); // 检查是否支持指定的字段
boolean isSupported1 = offsetDateTime.isSupported(ChronoUnit.DAYS); // 检查是否支持指定的时间单位
ValueRange valueRange = offsetDateTime.range(ChronoField.YEAR); // 获取指定字段的值范围
Comparator<OffsetDateTime> c = offsetDateTime.timeLineOrder(); // 获取时间线顺序
```
