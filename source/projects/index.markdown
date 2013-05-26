---
layout: page
title: "Projects"
date: 2013-05-26 14:52
comments: false
sharing: true
footer: true
---
### [ec2launcher](https://github.com/StudyBlue/ec2launcher)

* A powerful tool to help launch Amazon EC2 instances.
* Supports concepts of environments and applications.

### [metrics-statsd](https://github.com/organicveggie/metrics-statsd)

* Plugin for [codahale/metrics](https://github.com/codahale/metrics) to send metrics output to [Statsd](https://github.com/etsy/statsd/)

### [mongodb-graphite](https://github.com/organicveggie/mongodb-graphite)

* A Ruby client to retrieve statistics from a MongoDB cluster and push them to either Graphite or Statsd.
* Based on original work by [kamaradclimber](https://github.com/kamaradclimber/mongodb-graphite)
* Major code overhaul
* Expanded to include support for both Graphite and Statsd
* Documentation and CLI cleanup

# Chef Cookbooks

### [asgard](https://github.com/StudyBlue/chef-asgard](https://github.com/StudyBlue/chef-asgard)

*  Chef cookbook for installing [Netflix Asgard](https://github.com/Netflix/asgard).
* Installs and configures Netflix Asgard with Java 6 and Tomcat 7.
* Can also proxy via Apache.

### [mater](https://github.com/StudyBlue/chef-mater)

* Chef cookbook for installing [Mater](https://github.com/obfuscurity/mater), a bridge between [StatusBoard](http://panic.com/statusboard/) and [Graphite](http://graphite.wikidot.com/).
* Includes support for running Mater under [Unicorn](http://unicorn.bogomips.org/) with Apache as a proxy.

# Other Contributions

## Chef Cookbooks

### [chef-mongodb](https://github.com/edelight/chef-mongodb)

* [Disable automatic replica set and shard configuration](https://github.com/edelight/chef-mongodb/pull/101)

### [chef-tomcat](https://github.com/bryanwb/chef-tomcat)

* [Provide support to completely override context.xml](https://github.com/bryanwb/chef-tomcat/pull/9)


### [cookbook-elasticsearch](https://github.com/elasticsearch/cookbook-elasticsearch/)

* [Fix custom installation directory](https://github.com/elasticsearch/cookbook-elasticsearch/pull/80)

### [monit](https://github.com/apsoto/monit)

* [Additional configuration options as Chef attributes](https://github.com/apsoto/monit/pull/3)
* [Switch from library to definition](https://github.com/apsoto/monit/pull/4)


### [scala-sbt-cookbook](https://github.com/garrettux/scala-sbt-cookbook)

* [Add support for RedHat based OS](https://github.com/garrettux/scala-sbt-cookbook/pull/1)

## Other

### [bucky](https://github.com/cloudant/bucky)

* [Added support for Gauges](https://github.com/cloudant/bucky/pull/6)

### [codahale/metrics](https://github.com/codahale/metrics)

* [Fixed line platform-specific line endings](https://github.com/codahale/metrics/pull/197)
* [Support for using TimerContext with other classes](https://github.com/codahale/metrics/pull/237)

### [Logstash](http://logstash.net/)

* [Added support for AWS SQS queues for input/output](https://github.com/logstash/logstash/pull/211)


### [nagios-plugin-mongodb](https://github.com/mzupan/nagios-plugin-mongodb)

* [Fixed replication lag check and slave_ok deprecation warning](https://github.com/mzupan/nagios-plugin-mongodb/pull/24)

### [pychef](https://github.com/coderanger/pychef)

* [Workaround when FQDN not defined](https://github.com/coderanger/pychef/pull/6)