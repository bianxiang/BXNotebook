## 5. 新的日期和时间API

Java 1.0 had a Date class that was, in hindsight, unbelievably naïve, and had most of its methods deprecated in Java 1.1 when a Calendar class was introduced. Its API wasn’t stellar, its instances were mutable, and it didn’t deal with issues such as leap seconds. The third time is a charm, and the java.time API that is introduced in Java 8 has remedied the flaws of the past and should serve us for quite some time.

本章重点：

- 所有的java.time对象都是不可变的。
- `Instant`表示一个时点（类似于`Date`）。
- In Java time, each day has exactly 86,400 seconds (i.e., no leap seconds).
- `Duration`是两个`Instant`的差值。
- `LocalDateTime`没有时区信息。
- `TemporalAdjuster`方法处理创建的日历计算，如寻找某月第一个星期天
- `ZonedDateTime`是某个时区的一个时间点（类似于`GregorianCalendar`）。
- Use a `Period`, not a `Duration`, when advancing zoned time, in order to account for daylight savings time changes.
- 用`DateTimeFormatter`格式化和解析时间

### 5.1. The Time Line

Historically, the fundamental time unit, the second, was derived from Earth’s rotation around its axis. There are 24 hours or 24 × 60 × 60 = 86400 seconds in a full revolution, so it seems just a question of astronomical measurements to precisely define a second. Unfortunately, Earth wobbles slightly, and a more precise definition was needed. In 1967, a new precise definition of a second, matching the historical definition, was derived from an intrinsic property of atoms of caesium-133. Since then, a network of atomic clocks keeps the official time. Ever so often, the official time keepers synchronize the absolute time with the rotation of Earth. At first, the official seconds were slightly adjusted, but starting in 1972, “leap seconds” were occasionally inserted. (In theory, a second might need to be removed once in a while, but that has not yet happened.) There is talk of changing the system again. Clearly, leap seconds are a pain, and many computer systems instead use “smoothing” where time is artificially slowed down or sped up just before the leap second, keeping 86,400 seconds per day. This works because the local time on a computer isn’t all that precise, and computers are used to synchronizing themselves with an external time service.

The Java Date and Time API specification requires that Java uses a time scale that

- Has 86,400 seconds per day
- Exactly matches the official time at noon each day
- Closely matches it elsewhere, in a precisely defined way

That gives Java the flexibility to adjust to future changes in the official time. In Java, an `Instant` represents a point on the time line. An origin, called the epoch, is arbitrarily set at midnight of January 1, 1970 at the prime meridian that passes through the Greenwich Royal Observatory in London. This is the same convention used in the Unix/POSIX time. Starting from that origin, time is measured in 86,400 seconds per day, forwards and backwards, in nanosecond precision. The Instant values go back as far as a billion years (Instant.MIN). That’s not quite enough to express the age of the universe (around 13.5 billion years), but it should be enough for all practical purposes. After all, a billion years ago, the earth was covered in ice and populated by microsocopic ancestors of today’s plants and animals. The largest value, Instant.MAX, is December 31 of the year 1,000,000,000. The static method call Instant.now() gives the current instant. You can compare two instants with the equals and compareTo methods in the usual way, so you can use instants as timestamps. To find out the difference between two instants, use the static method Duration.between. For example, here is how you can measure the running time of an algorithm: Click here to view code image Instant start = Instant.now(); runAlgorithm();
Instant end = Instant.now(); Duration timeElapsed = Duration.between(start, end); long millis = timeElapsed.toMillis(); A Duration is the amount of time between two instants. You can get the length of a Duration in conventional units by calling toNanos, toMillis, toSeconds, toMinutes, toHours, or toDays. Durations require more than a long value for their internal storage. The number of seconds is stored in a long, and the number of nanoseconds in an additional int. If you want to make computations in nanosecond accuracy, and you actually need the entire range of a Duration, then you can use one of the methods in Table 5–1. Otherwise, you can just call toNanos and do your calculations with long values.



