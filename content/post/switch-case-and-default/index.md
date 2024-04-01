---
title: "Switch case and default"
date: "2019-01-13"
categories:
  - "Coding"
tags:
  - "1Z0-808"
  - "Java"
---

```java
int i = 0;

switch (i) {
    default:
        System.out.println("1");
    case 1:
        System.out.println("2");
    case 2:
        System.out.println("3");
}
```

This snippet will compile and run without issues. It might be dubious whether it's okay to put the "default" block at the first place, but you can place it anywhere. In this case, the result would be:

```
1
2
3
```

However there is a good practice to put the "default" block at the end of the switch statement. Or avoid using switch statements at all. 🙂
