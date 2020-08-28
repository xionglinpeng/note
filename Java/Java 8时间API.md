# Java 8时间API





## LocalDate

`java.time.LocalDate`



```java
public int get(TemporalField field)
public int getYear()
public int getMonthValue()
public Month getMonth()
public int getDayOfYear()
public int getDayOfMonth()
public DayOfWeek getDayOfWeek()
public IsoChronology getChronology()
public IsoEra getEra()
public long getLong(TemporalField field)
```



```java
public LocalDate plus(TemporalAmount amountToAdd)
public LocalDate plus(long amountToAdd, TemporalUnit unit)
public LocalDate plusYears(long yearsToAdd)
public LocalDate plusMonths(long monthsToAdd)
public LocalDate plusDays(long daysToAdd)
public LocalDate plusWeeks(long weeksToAdd)
```



```java
public LocalDate minus(TemporalAmount amountToSubtract)
public LocalDate minus(long amountToSubtract, TemporalUnit unit)
public LocalDate minusYears(long yearsToSubtract)
public LocalDate minusMonths(long monthsToSubtract)
public LocalDate minusDays(long daysToSubtract)
public LocalDate minusWeeks(long weeksToSubtract)
```



```java
public LocalDate with(TemporalAdjuster adjuster)
public LocalDate with(TemporalField field, long newValue)
public LocalDate withYear(int year)
public LocalDate withMonth(int month)
public LocalDate withDayOfYear(int dayOfYear)
public LocalDate withDayOfMonth(int dayOfMonth)
```



```java
public boolean isEqual(ChronoLocalDate other)
public boolean isAfter(ChronoLocalDate other)
public boolean isBefore(ChronoLocalDate other)
public boolean isSupported(TemporalField field)
public boolean isSupported(TemporalUnit unit)
public boolean isLeapYear()
```



```java
public LocalDateTime atTime(LocalTime time)
public LocalDateTime atTime(int hour, int minute)
public LocalDateTime atTime(int hour, int minute, int second)
public LocalDateTime atTime(int hour, int minute, int second, int nanoOfSecond)
public OffsetDateTime atTime(OffsetTime time)
```



```java
public long until(Temporal endExclusive, TemporalUnit unit)
public Period until(ChronoLocalDate endDateExclusive)
public Stream<LocalDate> datesUntil(LocalDate endExclusive)
public Stream<LocalDate> datesUntil(LocalDate endExclusive, Period step)
```



```java
public LocalDateTime atStartOfDay()
public ZonedDateTime atStartOfDay(ZoneId zone)
```



```java
public long toEpochDay()
public String format(DateTimeFormatter formatter)
public Temporal adjustInto(Temporal temporal)
public <R> R query(TemporalQuery<R> query)
public ValueRange range(TemporalField field)
```

## LocalTime

## LocalDateTime



plus

```java
public LocalDateTime plusDays(long days)
```















## Instant

`java.time.Instant`

```java
public int get(TemporalField field)
public long getLong(TemporalField field)
public long getEpochSecond()
public int getNano()
```





```java
public Instant plus(TemporalAmount amountToAdd)
public Instant plus(long amountToAdd, TemporalUnit unit)
public Instant plusSeconds(long secondsToAdd)
public Instant plusMillis(long millisToAdd)
public Instant plusNanos(long nanosToAdd)
```





```java
public Instant minus(TemporalAmount amountToSubtract)
public Instant minus(long amountToSubtract, TemporalUnit unit)
public Instant minusSeconds(long secondsToSubtract)
public Instant minusMillis(long millisToSubtract)
public Instant minusNanos(long nanosToSubtract)
```



```java
public boolean isAfter(Instant otherInstant)
public boolean isBefore(Instant otherInstant)
public boolean isSupported(TemporalField field)
public boolean isSupported(TemporalUnit unit)
```



```java
public Instant with(TemporalAdjuster adjuster)
public Instant with(TemporalField field, long newValue)
```



```java
public long toEpochMilli()
public <R> R query(TemporalQuery<R> query)
public ValueRange range(TemporalField field)
public Instant truncatedTo(TemporalUnit unit)
public Temporal adjustInto(Temporal temporal)
public OffsetDateTime atOffset(ZoneOffset offset)
public ZonedDateTime atZone(ZoneId zone)
```



```java
public long until(Temporal endExclusive, TemporalUnit unit)
```