Java 8通过发布新的Date-Time API (JSR 310)来进一步加强对日期与时间的处理。

在旧版的 Java 中，日期时间 API 存在诸多问题，其中有：

* 非线程安全 − java.util.Date 是非线程安全的，所有的日期类都是可变的，这是Java日期类最大的问题之一。

* 设计很差 − Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。

* 时区处理麻烦 − 日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。


在Java8中， 新的时间及⽇期API位于java.time包中， 该包中有哪些重要的类。 分别代表了什么？


`Instant`： 时间戳

`Duration`： 持续时间， 时间差

`LocalDate`： 只包含⽇期， ⽐如： 2016-10-20

`LocalTime`： 只包含时间， ⽐如： 23:12:10

`LocalDateTime`： 包含⽇期和时间， ⽐如： 2016-10-20 23:14:21

`Period`： 时间段

`ZoneOffset`： 时区偏移量， ⽐如： +8:00

`ZonedDateTime`： 带时区的时间

`Clock`： 时钟， ⽐如获取⽬前美国纽约的时间

新的java.time包涵盖了所有处理日期，时间，日期/时间，时区，时刻（instants），过程（during）与时钟（clock）的操作。

### LocalTime 和 LocalDate的区别？


`LocalDate`表⽰⽇期， 年⽉⽇， `LocalTime`表⽰时间， 时分
秒

### 获取当前时间

在Java8中，使用如下方式获取当前时间：
    
    LocalDate today = LocalDate.now();
    int year = today.getYear();
    int month = today.getMonthValue();
    int day = today.getDayOfMonth();
    System.out.printf("Year : %d Month : %d day : %d t %n", year,month, day);
    

### 创建指定日期的时间

    LocalDate date = LocalDate.of(2018, 01, 01);
    

### 检查闰年

直接使⽤LocalDate的isLeapYear即可判断是否闰年

    LocalDate nowDate = LocalDate.now();
    //判断闰年
    boolean leapYear = nowDate.isLeapYear();
    
### 计算两个⽇期之间的天数和⽉数

在Java 8中可以⽤java.time.Period类来做计算。

    Period period = Period.between(LocalDate.of(2018, 1, 5),LocalDate.of(2018, 2, 5));
    
