---
title: "Variables and fields instantiation"
date: "2019-01-11"
categories:
  - "Coding"
tags:
  - "1Z0-808"
  - "Java"
---

```java
package com.imsavva;

public class Test {
    private String testField;

    public static void main(String[] args) {
        String testVariable;
        System.out.println(testVariable);
    }
}
```

This code won't compile because of line 4:

When we declare a local variable, it must be initialized before use. Class fields don't have to be initialized. In case we didn't specify any value, a default value will be used.

The following list represents default field values:

```java
private byte b;     // 0
private short s;    // 0
private int i;      // 0
private long l;     // 0
private float f;    // 0.0f
private double d;   // 0.0
private char c;     // '\u0000'
private String str; // null
```
