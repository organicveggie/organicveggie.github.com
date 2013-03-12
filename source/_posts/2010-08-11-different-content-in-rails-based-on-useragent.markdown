---
layout: post
title: "Different content in Rails based on UserAgent"
date: 2010-08-11 10:57
comments: true
categories: 
 - Development
 - Programming
 - Rails
 - Ruby
 - Web Development
---
I was recently working on a website built using Rails that needed to render different content for certain user agents. Specifically, we needed simpler versions of certain pages for BlackBerry devices. Here's how I accomplished it.
<!-- more -->
First, I added a new mime-type for BlackBerry by adding the following line to `config/initializers/mime_types.rb`:

{% codeblock mime_types.rb lang:ruby %}
Mime::Type.register_alias "text/html", :blackberry
{% endcodeblock %}

Next, I added two utility methods to `app/controllers/application.rb`:

{% codeblock application.rb lang:ruby %}
# Checks UserAgent
def is_blackberry?
  ua = request.user_agent
  return false if ua.nil?
  return false if ! ua.downcase.index('blackberry')
 
  # Don't call the BlackBerry 9800 a BlackBerry, since it has a modern browser
  # based on WebKit:
  # Mozilla/5.0 (BlackBerry; U; BlackBerry 9800; en) AppleWebKit/534.1+ (KHTML, Like Gecko) Version/6.0.0.141 Mobile Safari/534.1+
  return false if ua.downcase.index('webkit')
 
  # Must be a BlackBerry!
  true
end
 
# Sets the respond_to format to blackberry if blackberry
def set_blackberry_format
  if !request.xhr? &amp;&amp; is_blackberry?
    request.format = :blackberry
  end
end
{% endcodeblock %}

With that in hand, it's easy to render BlackBerry specific content on specific pages:

{% codeblock lang:ruby %}
set_blackberry_format
respond_to do |format|
  format.blackberry
  format.html
  format.js { render :layout =&gt; false }
end
{% endcodeblock %}
