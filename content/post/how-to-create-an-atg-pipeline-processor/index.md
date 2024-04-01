---
title: "How to create an ATG pipeline processor?"
date: "2016-06-21"
categories:
  - "Coding"
tags:
  - "ATG"
  - "Java"
---

I'm currently studying Oracle ATG. Kinda big monstrous e-commerce platform. Today I had to create a commerce pipeline processor. So, I started googling and found this great tutorial by Oracle: [Creating processors](https://docs.oracle.com/cd/E24152_01/Platform.10-1/ATGCommProgGuide/html/s1705creatingprocessors01.html). Well, that's not bad. But it's a bit poor. For example, I'd like to know, what is the **Object pParam?** What is the **PipelineResult pResult?** Actually, there's the third (unanswered) question: why all **p**arameters starts with **"p"?**

Ok, I've found some answers to my questions. Let's see.

Firstly, at the time of writing this article, the Oracle doc said that we need to implement one method:

```java
int runProcess(Object pParam, PipelineResult pResult)
```

But actually **PipelineProcessor interface** has another method to implement, which is:

```java
int[] getRetCodes()
```

This method returns an array of integers that can be returned from **runProcess()** method.

My task was to show total items count in the order, when the order is submitted. It's a training task, as you remember, because I think there is a better way to do it instead of creating a pipeline processor.

Anyway, the first parameter of the runProcess() method is a HashMap (at least it was a HashMap in my case). This map contained some objects, and two two of them were interesting for me: Order and Request. I used them to get the items count and put it into the session.

The second parameter is used to store the error data for a pipeline execution, as mentioned here: [Interface PipelineResult JavaDoc](https://docs.oracle.com/cd/E26180_01/Platform.94/apidoc/atg/service/pipeline/PipelineResult.html).

I also extended my class from ApplicationLoggingImpl to use logging facilities. Here is the full example of the **ATG pipeline processor**:

```java
package com.imsavva.atg.pipeline;

public class OrderItemsCountPipelineProcessor extends ApplicationLoggingImpl implements PipelineProcessor {

    public int[] getRetCodes() {
        return new int[] { 1 };
    }

    public int runProcess(Object pParam, PipelineResult pResult) throws Exception {
        Map<String, Object> map = (Map<String, Object>) pParam;
        Order order = (Order) map.get("Order");
        HttpServletRequest request = (HttpServletRequest) map.get("Request");
        long count = order.getTotalReturnedItemsCount();

        request.getSession().setAttribute("orderItemsCount", count);

        if (isLoggingDebug()) {
            logDebug(String.format("Total order items count: %s", count));
        }

        return 1;
    }
}
```

To show items count on a JSP page, I used expression language:

```
Total items: ${orderItemsCount}
```
