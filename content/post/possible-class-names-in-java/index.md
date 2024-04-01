---
title: "Possible class names in Java"
date: "2019-01-15"
categories:
  - "Coding"
tags:
  - "1Z0-808"
  - "Java"
---

What would be the result of execution of the following code?

```java
package com.imsavva;

public class Test {
    public static void main(String[] args) {
        System.out.println(new _成分$().getGreeting());
    }
}

class _成分$ {
    public String getGreeting() {
        return "Hello";
    }
}
```

If your answer is "Hello", then you're right!  
You can use any characters from the Unicode charset, in any case, to name classes (as well as variables and methods), except some preserved words like "null", "volatile", "public", etc.

However, it's strongly recommended to stick with [the Java code conventions](https://www.oracle.com/technetwork/java/codeconventions-150003.pdf). The rules related to the class names are on page 10.

First two reasons to use the code conventions to me are:

- Readability – I would love to meet class names like "Eyjafjallajökull" from time to time. But, the rarer the better.
- Convenience for programmers from different countries – imagine the situation when you have to work with the code where class names contain symbols that are missing on your keyboard? 🙈
