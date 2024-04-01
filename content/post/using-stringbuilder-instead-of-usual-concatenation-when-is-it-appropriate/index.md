---
title: "Using StringBuilder instead of usual concatenation. When is it appropriate?"
date: "2017-12-21"
categories:
  - "Coding"
tags:
  - "Java"
  - "dig deeper"
---

You might know that using string concatenation in Java is not a good practice as it might affect performance. In this short article, I will try to describe when it is necessary to use StringBuilder and when we can afford using concatenation (+ sign).

Let's start with a simple example:

```java
package com.imsavva;

public class ConcatenationTestApp {

    public static void main(String[] args) {
        String str = "abc";
        str = str + "def";
        str += "12" + "34" + str;

        System.out.println(str);
    }
}
```

To see how it works under the hood, let's compile and decompile it. I tried this using Java 6 and Java 8.

Compiling the code:

```bash
javac src/com/imsavva/ConcatenationTestApp.java
```

Decompiling the class:

```bash
javap -c src/com/imsavva/ConcatenationTestApp
```

Continue reading to see what happens next.

We just decompiled our Java class. Let's see what's inside.

```java
Compiled from "ConcatenationTestApp.java"
public class com.imsavva.ConcatenationTestApp extends java.lang.Object{
public com.imsavva.ConcatenationTestApp();
  Code:
   0:   aload_0
   1:   invokespecial   #1; //Method java/lang/Object."<init>":()V
   4:   return

public static void main(java.lang.String[]);
  Code:
   0:   ldc     #2; //String abc
   2:   astore_1
   3:   new     #3; //class java/lang/StringBuilder
   6:   dup
   7:   invokespecial   #4; //Method java/lang/StringBuilder."<init>":()V
   10:  aload_1
   11:  invokevirtual   #5; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   14:  ldc     #6; //String def
   16:  invokevirtual   #5; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   19:  invokevirtual   #7; //Method java/lang/StringBuilder.toString:()Ljava/lang/String;
   22:  astore_1
   23:  new     #3; //class java/lang/StringBuilder
   26:  dup
   27:  invokespecial   #4; //Method java/lang/StringBuilder."<init>":()V
   30:  aload_1
   31:  invokevirtual   #5; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   34:  ldc     #8; //String 1234
   36:  invokevirtual   #5; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   39:  aload_1
   40:  invokevirtual   #5; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   43:  invokevirtual   #7; //Method java/lang/StringBuilder.toString:()Ljava/lang/String;
   46:  astore_1
   47:  getstatic       #9; //Field java/lang/System.out:Ljava/io/PrintStream;
   50:  aload_1
   51:  invokevirtual   #10; //Method java/io/PrintStream.println:(Ljava/lang/String;)V
   54:  return

}
```

Ta-da! As we can see in line 13 (3), the compiler used a StringBuilder instance to perform concatenation in the bytecode. **We don't need to use StringBuilder outside of loops if we don't want to use it.**

But the situation changes when we create a variable outside of a loop and then use concatenation inside it. Consider the following example:

```java
package com.imsavva;

public class ConcatenationInsideLoopTestApp {

    public static void main(String[] args) {
        String[] letters = new String[] { "a", "b", "c", "d", "e", "f" };
        String result = "Test-";

        for (int i = 0; i < letters.length; i++) {
            result += letters[i];
        }

        System.out.println(result);
    }
}
```

After compiling and decompiling this code as we did for the previous fragment, we got this:

```java
Compiled from "ConcatenationInsideLoopTestApp.java"
public class com.imsavva.ConcatenationInsideLoopTestApp extends java.lang.Object{
public com.imsavva.ConcatenationInsideLoopTestApp();
  Code:
   0:   aload_0
   1:   invokespecial   #1; //Method java/lang/Object."<init>":()V
   4:   return

public static void main(java.lang.String[]);
  Code:
   0:   bipush  6
   2:   anewarray       #2; //class java/lang/String
   5:   dup
   6:   iconst_0
   7:   ldc     #3; //String a
   9:   aastore
   10:  dup
   11:  iconst_1
   12:  ldc     #4; //String b
   14:  aastore
   15:  dup
   16:  iconst_2
   17:  ldc     #5; //String c
   19:  aastore
   20:  dup
   21:  iconst_3
   22:  ldc     #6; //String d
   24:  aastore
   25:  dup
   26:  iconst_4
   27:  ldc     #7; //String e
   29:  aastore
   30:  dup
   31:  iconst_5
   32:  ldc     #8; //String f
   34:  aastore
   35:  astore_1
   36:  ldc     #9; //String Test-
   38:  astore_2
   39:  iconst_0
   40:  istore_3
   41:  iload_3
   42:  aload_1
   43:  arraylength
   44:  if_icmpge       74
   47:  new     #10; //class java/lang/StringBuilder
   50:  dup
   51:  invokespecial   #11; //Method java/lang/StringBuilder."<init>":()V
   54:  aload_2
   55:  invokevirtual   #12; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   58:  aload_1
   59:  iload_3
   60:  aaload
   61:  invokevirtual   #12; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   64:  invokevirtual   #13; //Method java/lang/StringBuilder.toString:()Ljava/lang/String;
   67:  astore_2
   68:  iinc    3, 1
   71:  goto    41
   74:  getstatic       #14; //Field java/lang/System.out:Ljava/io/PrintStream;
   77:  aload_2
   78:  invokevirtual   #15; //Method java/io/PrintStream.println:(Ljava/lang/String;)V
   81:  return

}
```

StringBuilder is still used, right? But take a look how it is used! As we can see on line 58 (71), the loop starts from line 42 (41). The interesting thing happens on line 46 (47), **a new instance of StringBuilder is created every loop iteration!** You might find it a bit wasteful, that's why you shouldn't use concatenation in loops!

Let's see a good piece of code:

```java
package com.imsavva;

public class CorrectConcatenationInsideLoopTestApp {

    public static void main(String[] args) {
        String[] letters = new String[] { "a", "b", "c", "d", "e", "f" };
        StringBuilder builder = new StringBuilder("Test-");

        for (int i = 0; i < letters.length; i++) {
            builder.append(letters[i]);
        }

        System.out.println(builder.toString());
    }
}
```

Here we created an instance of the StringBuilder before the loop. Inside the loop, we invoke its `append` method. Let's decompile this piece of code:

```java
Compiled from "CorrectConcatenationInsideLoopTestApp.java"
public class com.imsavva.CorrectConcatenationInsideLoopTestApp extends java.lang.Object{
public com.imsavva.CorrectConcatenationInsideLoopTestApp();
  Code:
   0:   aload_0
   1:   invokespecial   #1; //Method java/lang/Object."<init>":()V
   4:   return

public static void main(java.lang.String[]);
  Code:
   0:   bipush  6
   2:   anewarray       #2; //class java/lang/String
   5:   dup
   6:   iconst_0
   7:   ldc     #3; //String a
   9:   aastore
   10:  dup
   11:  iconst_1
   12:  ldc     #4; //String b
   14:  aastore
   15:  dup
   16:  iconst_2
   17:  ldc     #5; //String c
   19:  aastore
   20:  dup
   21:  iconst_3
   22:  ldc     #6; //String d
   24:  aastore
   25:  dup
   26:  iconst_4
   27:  ldc     #7; //String e
   29:  aastore
   30:  dup
   31:  iconst_5
   32:  ldc     #8; //String f
   34:  aastore
   35:  astore_1
   36:  new     #9; //class java/lang/StringBuilder
   39:  dup
   40:  ldc     #10; //String Test-
   42:  invokespecial   #11; //Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
   45:  astore_2
   46:  iconst_0
   47:  istore_3
   48:  iload_3
   49:  aload_1
   50:  arraylength
   51:  if_icmpge       68
   54:  aload_2
   55:  aload_1
   56:  iload_3
   57:  aaload
   58:  invokevirtual   #12; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   61:  pop
   62:  iinc    3, 1
   65:  goto    48
   68:  getstatic       #13; //Field java/lang/System.out:Ljava/io/PrintStream;
   71:  aload_2
   72:  invokevirtual   #14; //Method java/lang/StringBuilder.toString:()Ljava/lang/String;
   75:  invokevirtual   #15; //Method java/io/PrintStream.println:(Ljava/lang/String;)V
   78:  return

}
```

Here, the StringBuilder instance is created on line 38 (36), the loop starts on line 45 (48). The code doesn't create any new instances of the StringBuilder inside the loop. Profit!

I hope this article gave you a better understanding of using concatenation and StringBuilder. 😊
