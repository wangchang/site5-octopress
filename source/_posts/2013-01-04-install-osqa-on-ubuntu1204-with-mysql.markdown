---
layout: post
title: "install osqa on ubuntu1204 with mysql"
date: 2013-01-04 16:58
comments: true
categories: 
---

adduser --home /home/osqa --shell /bin/bash osqa
http://wiki.osqa.net/display/docs/Ubuntu+with+Apache+and+mysql
wget http://www.osqa.net/releases/fantasy-island-0.9.0-beta3.tar.gz
sudo apt-get install apache2 libapache2-mod-wsgi

sudo rm /etc/apache2/sites-available/default\
    /etc/apache2/sites-available/default-ssl\
    /etc/apache2/sites-enabled/000-default

sudo vim /etc/apache2/sites-available/osqa
sudo apt-get install mysql-server mysql-client

