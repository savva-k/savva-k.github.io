---
title: "How to install Kibana as a Windows service"
date: "2017-07-14"
categories: ["DevOps"]
tags: ["Kibana", "Windows services", "Elasticsearch"]
image: images/main.png
---

Installing Elasticsearch as a service is an easy task:

```bash
cd elasticsearch_dir/bin elasticsearch-service.bat install
```

This creates a new service, which should be started manually.

However, I faced a problem installing Kibana 5.5.0 as a Windows service. The first solution I found didn't work.

```bash
sc create "Kibana 5.5.0" binPath= "path_to_kibana/bin/kibana.bat" depend="elasticsearch-service-x64"
```

If you got "Access denied" running this command, try running cmd.exe as Administrator. This command creates a service, but it doesn't start because of some errors. I've found a solution that helped me:

- Download the Non-Sucking Service Manager here [https://nssm.cc](https://nssm.cc)
- Change directory to nssm_dir/your_os and open a command line there
- Run `nssm install Kibana550`
- Choose kibana.bat as application path
- Select log files location on "I/O" tab (at least stdout and stderr)
- Enter your Elasticsearch service name on "Dependencies" tab
- Click "Install service"
- Open Windows Services and start your Kibana550 service

That worked for me, I hope it may help someone else. 😊✌️
