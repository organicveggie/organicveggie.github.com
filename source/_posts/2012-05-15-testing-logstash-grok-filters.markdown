---
layout: post
title: "Testing Logstash grok filters"
date: 2012-05-15 13:25
comments: true
categories:
 - Logstash
 - Ruby
 - Grok 
---
Logstash is an outstanding tool for collecting and parsing logfiles. In particular, the grok filter is extremely useful to extract specific pieces of data from your logfiles. Once you pull data out of the logfiles into fields, you can easily search on those fields. Unfortunately, I find the format for patterns in grok filter challenging to write correctly. If you mess up your pattern, you will end up with the dreaded "_grokparsefailure" tag on your log entry. What you need is a way to test your patterns before adding them to Logstash.
<!-- more -->
As it turns out, you can test your patterns using the grok ruby library. @a1cy demonstrated the basics to me, but I wanted to describe the full process.

## Requirements
* Ruby 1.9+
* jls-grok gem

On my machine, I chose to install Ruby 1.9.2 via RVM, but you can use whatever technique you want. If you try to use Ruby 1.8.7, you will get a syntax error when you load the grok gem.

## Load Grok
You want to use the "grok-pure" library, which is 100% ruby, rather than "grok", which depends on a C library.

{% highlight ruby %}
require 'rubygems'
require 'grok-pure'
grok = Grok.new
{% endhighlight %}

## Loading Patterns
Since the grok filter in Logstash depends heavily on pattern files, I recommend you download the standard patterns from Github. Either the standard set for Logstash:

[https://github.com/logstash/logstash/tree/master/patterns](https://github.com/logstash/logstash/tree/master/patterns)

Or you can use the ones with grok:

[https://github.com/jordansissel/ruby-grok/blob/master/patterns/pure-ruby](https://github.com/jordansissel/ruby-grok/blob/master/patterns/pure-ruby)

Dump these into a directory so that you can load them with grok.

{% highlight ruby %}
grok.add_patterns_from_file("/path/to/patterns/grok-patterns")
{% endhighlight %}

You can use the same technique to load your own pattern files. Or you can manually add patterns by name:

{% highlight ruby %}
grok.add_pattern("foo", ".*foo.*")
grok.add_pattern("bar", ".*bar.*")
{% endhighlight %}

## Testing Patterns
So, you have successfully loaded a set of patterns. How do you test something? Let's take a sample log line from a MongoDB logfile:

`Tue May 15 11:21:42 [conn1047685] moveChunk deleted: 7157`

Let's say we want to grab the date/time as well as the number of the deleted chunk. Unfortunately, the date/time format doesn't match any of the stock patterns. We can build up a new date pattern and test it each step of the way:

{% highlight ruby %}
text = "Tue May 15 11:21:42 [conn1047685] moveChunk deleted: 7157"
pattern = '%{DAY}'
grok.compile(pattern)
grok.match(text)
{% endhighlight %}

The last line returns false if the pattern fails to match the input text. If the pattern matches the input text, Grok.match() returns a Grok::Match object that contains a lot of useful information about the match. You can ask Grok to output the text it capture by using the captures() method:

{% highlight ruby %}
grok.match(text).captures()
 => {"DAY"=>["Tue"]}
{% endhighlight %}

You can also test named captures, the same way you would use them in the Logstash grok filter:

{% highlight ruby %}
pattern = '%{DAY:day_of_week}'
grok.compile(pattern)
grok.match(text).captures()
 => {"DAY:day_of_week"=>["Tue"]}
{% endhighlight %}

The full date/time pattern for this example looks like the following:

{% highlight ruby %}
pattern = '%{DAY} %{MONTH} %{MONTHDAY} %{TIME}'
grok.compile(pattern)
grok.match(text).capture()
 => {"DAY"=>["Tue"], "MONTH"=>["May"], "MONTHDAY"=>["15"], "TIME"=>["11:21:42"], "HOUR"=>["11"], "MINUTE"=>["21"], "SECOND"=>["42"]} 
{% endhighlight %}

Now let's say we want to re-use this date/time pattern and still extract the chunk id from the log line. We can add a named pattern to the grok object manually and then reuse it, the same way we used patterns from the files:

{% highlight ruby %}
grok.add_pattern("MONGO_TIMESTAMP", '%{DAY} %{MONTH} %{MONTHDAY} %{TIME}')
grok.compile("%{MONGO_TIMESTAMP}")
pattern = '%{MONGO_TIMESTAMP:timestamp}'
grok.compile(pattern)
grok.match(text).captures()
 => {"MONGO_TIMESTAMP:timestamp"=>["Tue May 15 11:21:42"], "DAY"=>["Tue"], "MONTH"=>["May"], "MONTHDAY"=>["15"], "TIME"=>["11:21:42"], "HOUR"=>["11"], "MINUTE"=>["21"], "SECOND"=>["42"]} 
{% endhighlight %}

We still want the chunk id, so we can use additional patterns:

{% highlight ruby %}
pattern = '%{MONGO_TIMESTAMP:timestamp}%{GREEDYDATA} moveChunk deleted: %{NUMBER:chunk_id}'
grok.compile(pattern)
grok.match(text).captures()
 => {"MONGO_TIMESTAMP:timestamp"=>["Tue May 15 11:21:42"], "DAY"=>["Tue"], "MONTH"=>["May"], "MONTHDAY"=>["15"], "TIME"=>["11:21:42"], "HOUR"=>["11"], "MINUTE"=>["21"], "SECOND"=>["42"], "GREEDYDATA"=>[" [conn1047685]"], "NUMBER:chunk_id"=>["7157"], "BASE10NUM"=>["7157"]}
{% endhighlight %}

Let's say we wanted to grab that connection number inside the square brackets:

{% highlight ruby %}
pattern = '%{MONGO_TIMESTAMP:timestamp} \\[conn%{NUMBER:connection}\\] moveChunk deleted: %{NUMBER:chunk_id}'
grok.compile(pattern)
grok.match(text).captures()
 => {"MONGO_TIMESTAMP:timestamp"=>["Tue May 15 11:21:42"], "DAY"=>["Tue"], "MONTH"=>["May"], "MONTHDAY"=>["15"], "TIME"=>["11:21:42"], "HOUR"=>["11"], "MINUTE"=>["21"], "SECOND"=>["42"], "NUMBER:connection"=>["1047685"], "BASE10NUM"=>["1047685", "7157"], "NUMBER:chunk_id"=>["7157"]} 
{% endhighlight %}

## Full Example
{% highlight ruby %}
require 'rubygems'
require 'grok-pure'
 
grok = Grok.new
 
text = "Tue May 15 11:21:42 [conn1047685] moveChunk deleted: 7157"
pattern = '%{MONGO_TIMESTAMP:timestamp} \\[conn%{NUMBER:connection}\\] moveChunk deleted: %{NUMBER:chunk_id}'
grok.compile(pattern)
grok.match(text).captures()
 => {"MONGO_TIMESTAMP:timestamp"=>["Tue May 15 11:21:42"], "DAY"=>["Tue"], "MONTH"=>["May"], "MONTHDAY"=>["15"], "TIME"=>["11:21:42"], "HOUR"=>["11"], "MINUTE"=>["21"], "SECOND"=>["42"], "NUMBER:connection"=>["1047685"], "BASE10NUM"=>["1047685", "7157"], "NUMBER:chunk_id"=>["7157"]} 
{% endhighlight %}

You can also find an ultra simple example in the grok example folder on Github: [https://github.com/jordansissel/ruby-grok/blob/master/examples/test.rb](https://github.com/jordansissel/ruby-grok/blob/master/examples/test.rb).