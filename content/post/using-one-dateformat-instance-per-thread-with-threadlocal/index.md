---
title: "Using one DateFormat instance per thread with ThreadLocal"
date: "2017-07-12"
categories:
  - "Coding"
tags:
  - "dates"
  - "concurrency"
  - "Java"
image: images/main.jpg
---

Here is a quick tip how to use DateFormat inside a ThreadLocal field. Why do we need that? The reason is the DateFormat class is not thread-safe but creating its instances is an expensive operation. So, this is kind of a workaround that creates only one instance of DateFormat per thread.

```java
package com.imsavva.test;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @author Savva Kodeikin
 */
public class ThreadLocalDateFormatTest {

    private static ThreadLocal<SimpleDateFormat> dateFormat = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("dd/MM/yyyy");
        }
    };

    public String formatDate(Date date) {
        return dateFormat.get().format(date);
    }
}
```
