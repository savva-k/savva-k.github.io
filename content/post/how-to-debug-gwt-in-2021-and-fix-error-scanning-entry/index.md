---
title: "Solving the issue with debugging GWT in Idea: Error scanning entry: module-info.class"
date: "2021-07-16"
categories: [ "Coding" ]
tags: [ "GWT", "Java", "Jetty" ]
image: images/main.png
---

Recently I had to debug a GWT application in Intellij Idea using the GWT plugin, but it seemed to be broken. I was getting the following issue when the app was starting:

```
Error scanning entry: module-info.class
```

Sooner or later I googled that the issue is in Jetty 9.2 which is used by the plugin to start applications. It was fixed in Jetty 9.4, so the following guide describes how to debug GWT apps via plugin using an external Jetty 9.4:

1. Download Jetty 9.4 (https://www.eclipse.org/jetty/download.php)
1. Extract it to any location you like, let's call it JETTY_PATH

The rest of the steps should be done in the Idea:

1. **Setup Jetty**
    1. Go to Settings (Ctrl + Alt + S) -> Application Servers
    1. Create a new server (press +, choose Jetty), specify JETTY_PATH as the path to the server
1. **Make sure you have an exploded war artifact created during the build**
    1. Go to the Project Structure (Ctrl + Alt + Shift + S) -> Artifacts
    1. You should have two artifacts there: your-project-name:war and your-project-name:exploded
    1. Select the exploded artifact and copy the value of the "Output Directory" property, we'll use it later as EXPLODED_WAR_PATH
    1. If you don't have those artifacts, add them using + menu
        
        Important: make sure that the exploded artifact has "your-project-name GWT compiler output" in the Output Layout tab
1. **Create a run configuration for Jetty**
    1. Go to Run -> Edit configurations
    1. Click on +, choose Jetty server -> Local, specify the following settings:

        * Server tab:

            + VM options: -Djetty.http.port=8095
        
            + Before launch:

                + Build
                + Build 'your-project-name:exploded' artifact

        * Deployment tab:
            
            + Deploy at the server startup: your-project-name:exploded
1. **Create a run configuration for the GWT plugin (GWT dev mode)**
    1. Go to Run -> Edit configurations
    1. Click on +, choose GWT configuration, specify the following settings:

        + Module: your-project-name
        + Dev mode parameters: -port 8095 -noserver -war EXPLODED_WAR_PATH
        + Working directory: /path/to/your-project-name
        + JRE: /path/to/JRE
        + Start page: App.html
1. **Run the Jetty configuration**
    + It could also be run in the debug mode if you have some back end part to debug
1. **Run the GWT configuration**

After these steps you should be able to debug the GWT app at http://localhost:8085/App.html

The Java code will be injected to the browser's page. It will be available via inspector, on the Source tab 🥴