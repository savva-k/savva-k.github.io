---
title: "Object initialization order"
date: "2015-06-29"
categories:
  - "Coding"
tags:
  - "interview questions"
  - "Java"
---

If you don't know how objects initialize in Java, it may be a surprise when you write some code. Click more to see Java object initialization order and an example.

The initialization order is:

1. Parent static initialize block
2. Parent static field
3. Child static field
4. Child static initialize block
5. Parent instance initialize block
6. Parent instance field
7. Parent constructor
8. Child instance field
9. Child instance initialize block
10. Child constructor

Static and instance initialize blocks execute sequentially.

```java
public class Test extends Parent {

    public static void main(String[] args) {
        new Child();
    }

    /**
     * Printing a string, when a static field is initializing
     *
     * @param s string value from Parent or Child
     * @return the same string
     */
    static String getString(String s) {
        System.out.println(s);
        return s;
    }

    /**
     * Printing a string, when a non-static field is initializing
     *
     * @param s
     *            string value from Parent or Child
     * @return the same string
     */
    static int getInt(String s) {
        System.out.println(s);
        return 0;
    }

}

class Parent {

    Parent() {
        System.out.println("Parent constructor, x = " + x); // 7
    }

    static {
        System.out.println("Parent static initializer"); // 1
    }

    {
        System.out.println("Parent dynamic instance initializer"); // 5
        x = 10;
    }

    static String s = Test.getString("Parent static instances"); // 2
    int x = Test.getInt("Parent object fields initialize"); // 6
}

class Child extends Parent {

    static String s = Test.getString("Child static instances"); // 3
    int x = Test.getInt("Child object fields initialize"); // 8

    static {
        System.out.println("Child static initializer"); // 4
    }

    {
        System.out.println("Child dynamic instance initializer"); // 9
        x = 10;
    }

    Child() {
        System.out.println("Parent constructor, x = " + x); // 10
    }
}
```

After running this code you will see:

```
Parent static initializer
Parent static instances
Child static instances
Child static initializer
Parent dynamic instance initializer
Parent object fields initialize
Parent constructor, x = 0
Child object fields initialize
Child dynamic instance initializer
Parent constructor, x = 10
```
