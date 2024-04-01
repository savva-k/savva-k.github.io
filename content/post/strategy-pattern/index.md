---
title: "Strategy pattern"
date: "2016-06-12"
categories:
  - "Coding"
tags:
  - "patterns"
  - "Java"
---

With this post I'm starting series of articles dedicated to GoF design patterns. GoF stands for Gang of Four - four authors that wrote a famous book about design patterns.

So the first pattern I want to describe is **Strategy**. It allows to choose the behaviour of instances at runtime by defining appropriate strategy.

Let's see an example. Supposing we have a class **Order**, which has **getTotal()** method, that returns total cost of **Items** in this order:

```java
public int getTotal() {
    int total = 0;

    for (Item item : items) {
        total += item.getPrice();
    }

    return total;
}
```

Very simple, isn't it? But what if we want to change the price depending on some circumstances? That's where we can use the **Strategy pattern**. Click read more to see full example.

First, let's create the PriceStrategy interface. The only method it has receives a price of an item, changes it depending on pricing strategy and returns it:

```java
package patterns.strategy;

public interface PriceStrategy {

    /**
     * Calculate an item price.
     *
     * @param price to handle
     * @return item price
     */
    int calculatePrice(int price);
}
```

Now let's implement this interface. Normally we don't want to change the price, thus the NormalPriceStrategy class would look like:

```java
package patterns.strategy;

public class NormalPriceStrategy implements PriceStrategy {

    @Override
    public int calculatePrice(int price) {
        return price;
    }

}
```

Sometimes shops offer us 50% discount, so we need to provide the same pricing policy in our application:

```java
package patterns.strategy;

public class HalfPriceStrategy implements PriceStrategy {

    @Override
    public int calculatePrice(int price) {
        return price / 2;
    }

}
```

And why not to give our customers additional 10% discount on Mondays? Let's do it:

```java
package patterns.strategy;

import java.util.Calendar;

public class MondayTenPercentDiscountPriceStrategy implements PriceStrategy {

    private static final double MONDAY_DISCOUNT_COEF = 0.9;

    @Override
    public int calculatePrice(int price) {
        Calendar calendar = Calendar.getInstance();

        if (Calendar.MONDAY == calendar.get(Calendar.DAY_OF_WEEK)) {
            price = (int) (price * MONDAY_DISCOUNT_COEF);
        }

        return price;
    }

}
```

Now we have 3 different pricing strategies. All we need is to use one of them in the Order class. Let's define a **priceStrategy** property of type **PriceStrategy**, create getter and setter methods, and, optionally, create a constructor which receives some strategy. The main idea is to use strategy's **calculatePrice()** method while calculating total order cost:

```java
package patterns.strategy;

import java.util.List;

public class Order {

    private List<Item> items;
    private PriceStrategy pricingStrategy;

    public Order() {

    }

    public Order(PriceStrategy strategy, List<Item> items) {
        this.pricingStrategy = strategy;
        this.items = items;
    }

    public PriceStrategy getStrategy() {
        return pricingStrategy;
    }

    public void setStrategy(PriceStrategy strategy) {
        this.pricingStrategy = strategy;
    }

    public List<Item> getItems() {
        return items;
    }

    public void setItems(List<Item> items) {
        this.items = items;
    }

    public int getTotal() {
        int total = 0;

        for (Item item : items) {
            total += pricingStrategy.calculatePrice(item.getPrice());
        }

        return total;
    }
}
```

Notice that we've changed this line

```java
total += item.getPrice();
```

To this one:

```java
total += strategy.calculatePrice(item.getPrice());
```

Now let's write a simple main method to run the application and try to use all of our strategies:

```java
package patterns.strategy;

import java.util.ArrayList;
import java.util.List;

public class StrategyPatternApp {

    public static void main(String[] args) {
        Order order = new Order(new NormalPriceStrategy(), getItems());

        System.out.println("Total price (normal strategy): " + order.getTotal());

        order.setStrategy(new HalfPriceStrategy());

        System.out.println("Total price (half price strategy): " + order.getTotal());

        order.setStrategy(new MondayTenPercentDiscountPriceStrategy());

        System.out.println("Total price (Monday 10% off price strategy): " + order.getTotal());
    }

    private static List<Item> getItems() {
        List<Item> items = new ArrayList<>();

        items.add(new Item("A travel guide book", 10));
        items.add(new Item("A tent", 70));
        items.add(new Item("A sleeping bag", 35));

        return items;
    }
}
```

Run the app. The result will be:

```
Total price (normal strategy): 115
Total price (half price strategy): 57
Total price (Monday 10% off price strategy): 103
``` 

Of course, the third result will be different if you run this application not on Monday. I'm sure you will like this pattern, as it makes your code more flexible. I suggest you to read an article about this pattern in [Wikipedia](https://en.wikipedia.org/wiki/Strategy_pattern).
