---
layout: post
title: "Distributed Graphite on EC2"
date: 2012-05-28 14:20
comments: true
categories: 
 - Metrics
 - Graphite
 - AWS
 - EC2 
---
Some time ago, we switched from [Cacti](http://www.cacti.net/) to [Graphite](http://graphite.wikidot.com/) for tracking and graphing system metrics. In our [Amazon EC2 environment](http://aws.amazon.com/ec2/), we frequently startup new servers and shut down old servers. While Cacti did a decent job, the amount of manual effort required to setup new graphs made it a challenge to use. Since Graphite simply tracks anything you throw at it and easily handles applying aggregate functions across multiple metrics, we found Graphite to be a much better fit with EC2 and our usage patterns. With a solid web interface and a variety of alternate front-ends and awesome dashboard tools like [Graphene](http://jondot.github.com/graphene/), we quickly fell in love with Graphite.
<!-- more -->
In fact, the more we utilized Graphite, the more data we wanted to push into Graphite. Systems metrics? Check. IO stats? Check. Apache stats? Check. MongoDB stats on a per collection basis? Why not! We quickly found ourselves with over 50,000 metrics and 500GB of data, all of which was getting updated every 15-30 seconds. Unfortunately, we also rapidly hit the limit of how much data we could record on a single instance. Even with a RAID-0 array of 8x EBS volumes on an m2.2xlarge instance, we couldn't write the data fast enough. The poor instance began spending more than 50% of it's time in I/O wait  and eventually became non-responsive. What we needed was a way to distribute the write load across multiple instances.

Fortunately, Graphite natively supports running in a distributed environment. A successful deployment consists of three key pieces:

* carbon-cache - This is what actually accepts and stores the metrics.
* carbon-relay - Handles sharding the metrics and sending them to the appropriate carbon-cache instances.
* webapp - Handles displaying the metrics. Must reside on every instance running a carbon-cache daemon. The webapp queries the other carbon-cache instances for data as necessary.

Although there are many different ways you can setup your distributed Graphite environment, we chose to take a straight foward approach:

* One dedicated instance running carbon-relay that receives metrics from the various server instances, shards the metrics and forwards them on to the carbon-cache servers
* Two dedicated instances running both carbon-cache and webapp.

## Carbon Relay

On the carbon-relay server, you need to configure two key files to indicate that this node should run as a relay: carbon.conf and relay.conf.

`carbon.conf`

{% highlight properties %}
[relay]
RELAY_METHOD = consistent-hashing

LINE_RECEIVER_INTERFACE = x.x.x.x
LINE_RECEIVER_PORT = 2003

PICKLE_RECEIVER_INTERFACE = 127.0.0.1
PICKLE_RECEIVER_PORT = 2004

DESTINATIONS = cache1:2004:a,cache2:2004:a

MAX_DATAPOINTS_PER_MESSAGE = 500
MAX_QUEUE_SIZE = 10000

USE_FLOW_CONTROL = True 
{% endhighlight %}

For simplicity, we're using the consistent-hashing method of sharding. The advantage to consistent-hasing is that Graphite will automatically split the metrics evenly across all carbon-cache instances. If you add another carbon-cache node, Graphite will start sending data to it. The disadvantage is that you have absolutely no control over which metric goes to which server. If you start with two nodes and then add a third node, Graphite will not automatically rebalance the data.

A couple key things to note:

* The RELAY_METHOD value is set to consistent-hashing
* The LINE_RECEIVER_INTERFACE must be set to the IP address of the relay node, not localhost
* The DESTINATIONS value must contain all of the target carbon-cache nodes
* Each server listed in the DESTINATIONS option must include the name or ip address of a carbon-cache instance, the pickle-receiver port for the remote instance and the carbon-cache identifier. If you run multiple carbon-cache daemons on the remote server,  the first instance will be identified as "a", the second instance as "b", etc.

`relay.conf`

{% highlight properties %}
[default]
default = true
destinations = cache1,cache2
{% endhighlight %}

This sample relay.conf shows us splitting the data automatically across two servers: cache1 and cache2.

## Carbon Cache

On the carbon-cache servers, we need to configure the carbon-cache daemon to appropriately accept incoming data from the carbon-relay servers.

`carbon.conf`

{% highlight properties %}
[cache]
LOCAL_DATA_DIR = /opt/graphite/storage/whisper/

MAX_CACHE_SIZE = inf

MAX_UPDATES_PER_SECOND = 800

MAX_CREATES_PER_MINUTE = 100

LINE_RECEIVER_INTERFACE = y.y.y.y
LINE_RECEIVER_PORT = 2003

PICKLE_RECEIVER_INTERFACE = y.y.y.y
PICKLE_RECEIVER_PORT = 2004

CACHE_QUERY_INTERFACE = 127.0.0.1
CACHE_QUERY_PORT = 7002

LOG_UPDATES = False
{% endhighlight %}

There are several critical settings here. First, the pickle receiver must be configured correctly to allow incoming data from the carbon-relay server. Second, the line receiver must be configured correctly to allow the the webapp to communicate with the other carbon-cache instances.

* LINE_RECEIVER_INTERFACE must be set to the local ip address (not 127.0.0.1)
* LINE_RECEIVER_PORT must be set to 2003
* PICKLE_RECEIVER_INTERFACE must be set to the local ip address (not 127.0.0.1)
* PICKLE_RECEIVER_PORT must match the port we listed in the DESTINATION line in carbon.conf on the carbon-relay server

The other key is configuring the local_settings.py config file for the webapp correctly:

{% highlight properties %}
CLUSTER_SERVERS = [ 'cache1','cache2' ]
MEMCACHE_HOSTS = [ 'memcache1.graphite:11211' ]
{% endhighlight %}

The only mandatory piece of information in local_settings.py is CLUSTER_SERVERS, which must be a Python array containing strings that are the names (or ip address) of every carbon-cache server. You can optionally include an array of memcached instances to improve web performance dramatically. If you choose to utilize memcached, you can either run it locally on each carbon-cache instance or you can run a cluster of one or more memcached servers remotely. In the example above, we're using a single memcached instance running on a remote server.

IMPORTANT NOTE: The local_settings.py file MUST be readable by the web server user. If you're running Apache, this is typically the apache user. If the local_settings.py file is NOT readable, you will get very very strange errors.

## Firewall Rules

To get this all working, specific ports must be open on the various firewalls. If you're using Amazon EC2, your security groups will need to contain rules explicitly allowing incoming traffic on these ports:

### carbon-relay
1. Incoming on port 2003

### carbon-cache/webapp

1. Incoming on port 2004 (messages from carbon-relay)
1. Incoming on port 2003 (requests for data from other carbon-cache servers)
1. Incoming on port 80 (web requests)
1. Incoming on port 11211 (if you're running memcached locally)

