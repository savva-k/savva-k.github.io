---
title: "Observer pattern"
date: "2016-06-19"
categories:
  - "Coding"
tags:
  - "patterns"
  - "Java"
---

**Observer** is a software design pattern, that is used when some objects create events and other objects should be notified when these events occur.

The main idea is that we have two types of instances: one that produces events and one that consumes them. Events producer must have three methods:

1. addObserver(Observer observer)
2. removeObserver(Observer observer)
3. notifyObservers(...)

The first two methods are used to add and delete observers from a collection. The third method must iterate that collection and call notify(...) method of the Observer instance.

Java has its own Observer interface and Observable class, but sometimes we don't want to extend from a class, or we want to adjust the API, you can create your own interfaces, as I've done in the following example:

Observer interface:

```java
package com.imsavva.tests.patterns.observer;

public interface Observer {
    void notify(String message);
}
```

Observable interface:

```java
package com.imsavva.tests.patterns.observer;

public interface Observable {
    void addObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers(String message);
}
```

Observer implementation (let it be a ship that sails in the Carribean sea)

```java
package com.imsavva.tests.patterns.observer;

public class Ship implements Observer {

    private String name;

    public Ship(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public void notify(String message) {
        System.out.println(String.format("Ship %s received a message: %s", name, message));
    }

}
```

And the Observable interface implementation (this is the sea port which sends radio messages to all the ships)

```java
package com.imsavva.tests.patterns.observer;

import java.util.ArrayList;
import java.util.List;

public class SeaPort implements Observable {

    private List<Observer> observers = new ArrayList<>();

    @Override
    public void addObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers(String message) {
        for (Observer observer : observers) {
            observer.notify(message);
        }
    }

    public void start() {
        for (int i = 0; i < 5; i++) {
            String message = getRandomHelpMessage();
            notifyObservers(message);
        }
    }

    private String getRandomHelpMessage() {
        int x = (int) (Math.random() * 10);
        int y = (int) (Math.random() * 10);

        return String.format("Some people need help at point [%s, %s]", x, y);
    }

}
``` 

And here's the main method that runs a test app:

```java
package com.imsavva.tests.patterns.observer;

public class ObserverPatternApp {

    public static void main(String[] args) {
        SeaPort port = new SeaPort();
        Ship ship1 = new Ship("Jenny");
        Ship ship2 = new Ship("Nebuchadnezzar");

        port.addObserver(ship1);
        port.addObserver(ship2);

        port.start();
    }


}
```

When we run the application, we will see something like this. All the subscribed ships (observers) have received the messages and they're hurrying to help the people. Of course it would be nice if captains negotiated with each other whom to help. But that's another story.

```
Ship Jenny received a message: Some people need help at point [3, 6]
Ship Nebuchadnezzar received a message: Some people need help at point [3, 6]
Ship Jenny received a message: Some people need help at point [2, 7]
Ship Nebuchadnezzar received a message: Some people need help at point [2, 7]
Ship Jenny received a message: Some people need help at point [7, 8]
Ship Nebuchadnezzar received a message: Some people need help at point [7, 8]
Ship Jenny received a message: Some people need help at point [4, 5]
Ship Nebuchadnezzar received a message: Some people need help at point [4, 5]
Ship Jenny received a message: Some people need help at point [4, 1]
Ship Nebuchadnezzar received a message: Some people need help at point [4, 1]
```

Using this pattern can lead to the "lapsed listener problem". It's a common source of memory-leaks in OOP. This problem occurs when an observer no longer needs to listen to a message source, and it fails to unsubscribe. Thus, it cannot be garbage collected as the message source instance holds the reference to the observer. This problem can be solved by using WeakReference. And I think I will write about it in one of my next posts.
