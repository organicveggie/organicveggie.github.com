---
layout: post
title: "Asgard and Mater Cookbooks"
date: 2013-04-29 21:00
comments: true
categories: 
 - Chef
---
Over at [StudyBlue](http://www.studyblue.com), we leverage [Chef](http://www.opscode.com/chef/) pretty heavily for automation. Today we just open sourced two Chef cookbooks as a way to give back to the community that has supported us.
<!-- more -->
## Asgard
[https://github.com/StudyBlue/chef-asgard](https://github.com/StudyBlue/chef-asgard)

*  Chef cookbook for installing [Netflix Asgard](https://github.com/Netflix/asgard).
* Installs and configures Netflix Asgard with Java 6 and Tomcat 7.
* Can also proxy via Apache.

## Mater
[https://github.com/StudyBlue/chef-mater](https://github.com/StudyBlue/chef-mater)

* Chef cookbook for installing [Mater](https://github.com/obfuscurity/mater), a bridge between [StatusBoard](http://panic.com/statusboard/) and [Graphite](http://graphite.wikidot.com/).
* Includes support for running Mater under [Unicorn](http://unicorn.bogomips.org/) with Apache as a proxy.