---
layout: post
title: "DRY Associations in Rails"
date: 2007-08-19 23:00
comments: true
categories: 
 - Rails
 - DRY
---
I found a nice little article from a few months ago on [The Rails Way](http://therailsway.com/). Written by koz, [Assocation Proxies](http://therailsway.com/2007/3/26/association-proxies-are-your-friend) talks about some Best Practices for using assocations within Rails. I really felt the suggestions hit the proverbial nail on the head in terms of following the DRY principles. The first suggestion was on the best way to restrict access to user specific information. It's incredibly useful to take advantage your RESTful patterns and make a single call like:

{% codeblock lang:ruby %}
@todo_list = current_user.todo_lists.find(params[:id])
{% endcodeblock %}

Of course, I also loved the final suggestion in Case Three: using a has_many with a condition on it:

{% codeblock lang:ruby %}
has_many :completed_lists, :class_name => "TodoList", :conditions => { :completed => true }
{% endcodeblock %}

Excellent!