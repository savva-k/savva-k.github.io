---
title: "How to lookup a user profile from a global-scoped ATG component"
date: "2017-06-30"
categories:
  - "Coding"
tags:
  - "ATG"
  - "Java"
image: images/main.jpg
---

This article describes how to look up a session-scoped user profile component from a global scope component **that has no access** to a DynamoHttpServletRequest object. To achieve this functionality, we will use a ThreadLocal field an ATG feature called [Insertable Servlet](https://docs.oracle.com/cd/E24152_01/Platform.10-1/ATGPlatformProgGuide/html/s0807insertingservletsinthepipeline01.html). When a request object is created, it goes through a pipeline of Insertable Servlets. We are going to create our own implementation of Insertable Servlet by extending _atg.servlet.pipeline.InsertableServletImpl_ which will store current request to a ThreadLocal variable via a helper class (e.g. RequestContainer). Then, we're going to inject this RequestContainer in every component that needs to look up the profile (or other components).

Here is the code.

### 1. Creating a request container

Java code

```java
public class RequestContainer extends GenericService {

    ThreadLocal<DynamoHttpServletRequest> requestContainer;

    @Override
    public void doStartService() throws ServiceException {
        super.doStartService();
        requestContainer = new ThreadLocal<DynamoHttpServletRequest>();
    }

    public Object resolveName(String nucleusName) {
        DynamoHttpServletRequest request = getRequest();
        if(request == null) {
            return null;
        } else {
            return request.resolveName(nucleusName);
        }
    }

    public DynamoHttpServletRequest getRequest() {
        return requestContainer.get();
    }

    public void setRequest(DynamoHttpServletRequest request) {
        requestContainer.set(request);
    }
}
```

ATG properties file

```properties
# /com/imsavva/atg/misc/RequestContainer
$class=com.imsavva.atg.misc.RequestContainer
```

### 2. Creating an Insertable Servlet

```java
package com.imsavva.atg.pipeline;

// imports are omitted

public class RequestContainerSetterServlet extends InsertableServletImpl {

    private RequestContainer requestContainer;

    @Override
    public void service(DynamoHttpServletRequest pRequest, DynamoHttpServletResponse pResponse) throws IOException, ServletException {
        requestContainer.setRequest(pRequest);

        try {
            passRequest(pRequest, pResponse);
        } finally {
            requestContainer.setRequest(null);
        }
    }

    public RequestContainer getRequestContainer() {
        return requestContainer;
    }

    public void setRequestContainer(RequestContainer requestContainer) {
        this.requestContainer = requestContainer;
    }
}
```

Notice, that in line 16 we’re removing our request as it is already processed:

```java
requestContainer.setRequest(null);
```

```properties
# /com/imsavva/atg/pipeline/RequestContainerSetterServlet
$class=com.imsavva.atg.pipeline.RequestContainerSetterServlet

insertAfterServlet=/atg/dynamo/servlet/dafpipeline/DynamoHandler
requestContainer=/com/imsavva/atg/misc/RequestContainer
```

### 3. Adding the servlet's component to initial services

To start our Initial Servlet with the application, let's add this line to the /atg/dynamo/servlet/Initial component configuration file:

```properties
initialServices+=/com/imsavva/atg/pipeline/RequestContainerSetterServlet
```

### 4. Testing our workaround

To test our workaround let's create a new component, inject the request container into it and try to lookup the user's profile.

```java
public class LookupTestComponent extends GenericService {
    private RequestContainer requestContainer;

    public void tryLookup() {
        Profile profile = (Profile) getRequestContainer().resolveName("/atg/userprofiling/profile");
        String lastName = (String) profile.getPropertyValue("lastName");
        logInfo("Got user: " + lastName);
    }

    public RequestContainer getRequestContainer() {
        return requestContainer;
    }

    public void setRequestContainer(RequestContainer requestContainer) {
        this.requestContainer = requestContainer;
    }
}
```

Configuration file for the component

```properties
# /com/imsavva/atg/LookupTestComponent
$class=com.imsavva.atg.LookupTestComponent

requestContainer=/com/imsavva/atg/misc/RequestContainer
```

That's it. Now we can try to log in and invoke the "tryLookup()" method. We'll see that the profile was resolved correctly.

### Instead of conclusion

Love Spring Framework 👾
