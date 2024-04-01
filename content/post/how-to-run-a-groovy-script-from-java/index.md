---
title: "How to run a Groovy script from Java"
date: "2017-03-01"
categories:
  - "Coding"
tags:
  - "Groovy"
  - "Java"
---

One of the ways of running a Groovy script from Java is using GroovieShell. First, add the groovy-all Maven dependency (consider using a fresher version):

```xml
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-all</artifactId>
    <version>2.1.5</version>
</dependency>
```

Next, create a class containing the main method. The Binding instance helps us to bind variables to the shell. These variables will be available for our script. We can use all types of variables, not only String as in the example.

```java
public class App {

    public static void main(String[] args) throws CompilationFailedException, IOException {
        Binding binding = new Binding();
        GroovyShell shell = new GroovyShell(binding);

        binding.setVariable("words", "world hello");
        Object response = shell.evaluate(new File("src/main/groovy/wordSwapper.groovy"));

        System.out.println(String.valueOf(response));
    }
}
```

As you can see from the code above, I mentioned a test Groovy script. It's pretty small. Let's create it in `src/main/groovy/wordSwapper.groovy`

```groovy
def wordsArray = words.split(' ')

wordsArray[1] + ' ' + wordsArray[0]
```

That's it. After launching the application you'll see that our script transformed string "world hello" to "hello world".
