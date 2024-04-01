---
title: "Silent installation MySQL 5.7 on Ubuntu"
date: "2016-04-17"
categories: ["DevOps"]
tags: ["Bash", "Linux", "MySQL", "Ubuntu"]
image: "images/main.png"
---

This week I had to install MySQL 5.7 using a shell script. So maybe it would be helpful for someone else. This script must be run as root.

```bash
ROOT_PASSWORD="your_root_password"

echo "mysql-apt-config mysql-apt-config/unsupported-platform select abort" | /usr/bin/debconf-set-selections
echo "mysql-apt-config mysql-apt-config/repo-codename   select trusty" | /usr/bin/debconf-set-selections
echo "mysql-apt-config mysql-apt-config/select-tools select" | /usr/bin/debconf-set-selections
echo "mysql-apt-config mysql-apt-config/repo-distro select ubuntu" | /usr/bin/debconf-set-selections
echo "mysql-apt-config mysql-apt-config/select-server select mysql-5.7" | /usr/bin/debconf-set-selections
echo "mysql-apt-config mysql-apt-config/select-product select Apply" | /usr/bin/debconf-set-selections

echo "mysql-community-server mysql-community-server/root-pass password $ROOT_PASSWORD" | /usr/bin/debconf-set-selections
echo "mysql-community-server mysql-community-server/re-root-pass password $ROOT_PASSWORD" | /usr/bin/debconf-set-selections
echo "mysql-community-server mysql-community-server/remove-data-dir boolean false" | /usr/bin/debconf-set-selections
echo "mysql-community-server mysql-community-server/data-dir note" | /usr/bin/debconf-set-selections

export DEBIAN_FRONTEND=noninteractive
wget http://dev.mysql.com/get/mysql-apt-config_0.6.0-1_all.deb
dpkg --install mysql-apt-config_0.6.0-1_all.deb
apt-get update
apt-get --yes --force-yes install mysql-server-5.7
```

If you want to install software automatically, you may be wondering how to know which parameters to set. How to do that?

Use `debconf-get-selections`! To install it run:

```bash
sudo apt-get install -y debconf-utils
```

To find properties related to MySQL, run:

```bash
debconf-get-selections | grep mysql
```

Then you can set it in your script like this:

```bash
echo "mysql-community-server mysql-community-server/root-pass password $ROOT_PASSWORD" | /usr/bin/debconf-set-selections
```
