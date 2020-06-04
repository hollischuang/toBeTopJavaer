了解Java8 的朋友可能都知道，Java8提供了一套新的时间处理API，这套API比以前的时间处理API要友好的多。

Java8 中加入了对时区的支持，带时区的时间为分别为：`ZonedDate`、`ZonedTime`、`ZonedDateTime`。

其中每个时区都对应着 ID，地区ID都为 “{区域}/{城市}”的格式，如`Asia/Shanghai`、`America/Los_Angeles`等。

在Java8中，直接使用以下代码即可输出美国洛杉矶的时间：
    
    LocalDateTime now = LocalDateTime.now(ZoneId.of("America/Los_Angeles"));
    System.out.println(now);
    
    
    
为什么以下代码无法获得美国时间呢？

    System.out.println(Calendar.getInstance(TimeZone.getTimeZone("America/Los_Angeles")).getTime());
    
当我们使用System.out.println来输出一个时间的时候，他会调用Date类的toString方法，而该方法会读取操作系统的默认时区来进行时间的转换。
    
    public String toString() {
        // "EEE MMM dd HH:mm:ss zzz yyyy";
        BaseCalendar.Date date = normalize();
        ...
    }
    
    private final BaseCalendar.Date normalize() {
        ...
        TimeZone tz = TimeZone.getDefaultRef();
        if (tz != cdate.getZone()) {
            cdate.setZone(tz);
            CalendarSystem cal = getCalendarSystem(cdate);
            cal.getCalendarDate(fastTime, cdate);
        }
        return cdate;
    }
    
    static TimeZone getDefaultRef() {
        TimeZone defaultZone = defaultTimeZone;
        if (defaultZone == null) {
            // Need to initialize the default time zone.
            defaultZone = setDefaultZone();
            assert defaultZone != null;
        }
        // Don't clone here.
        return defaultZone;
    }
    
主要代码如上。也就是说如果我们想要通过`System.out.println`输出一个Date类的时候，输出美国洛杉矶时间的话，就需要想办法把`defaultTimeZone`改为`America/Los_Angeles`
    
但是，通过阅读Calendar的源码，我们可以发现，getInstance方法虽然有一个参数可以传入时区，但是并没有将默认时区设置成传入的时区。

而在Calendar.getInstance.getTime后得到的时间只是一个时间戳，其中未保留任何和时区有关的信息，所以，在输出时，还是显示的是当前系统默认时区的时间。