---
title: "Parsing a date string to ISO8601 with Joda Time"
date: "2017-02-26"
categories:
  - "Coding"
tags:
  - "dates"
  - "Java"
---

Recently, I faced a date conversion task: convert a string date "yyyy-MM-ddZ" (i.e. "1983-09-15+03:00") to the ISO8601 standard "yyyy-MM-dd'T'HH:mm:ss.SSSZ" (i.e. "1983-09-15T03:00:00.000+03:00").

I used Apache Joda time to convert date. First, add a Maven dependency in pom.xml

```xml
<!-- https://mvnrepository.com/artifact/joda-time/joda-time -->
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.9.4</version>
</dependency>
```

To format a string containing date, I used this code:

```java
DateTimeFormatter format = DateTimeFormat.forPattern("yyyy-MM-ddZ");
DateTime dt = format.parseDateTime("1983-09-15+03:00")
.withZone(DateTimeZone.forOffsetHours(3))
.withTime(3, 0, 0, 0);

System.out.println(dt);
```

The result in a console will be `1983-09-15T03:00:00.000+03:00`
