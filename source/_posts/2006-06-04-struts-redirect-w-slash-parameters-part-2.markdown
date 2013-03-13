---
layout: post
title: "Struts Redirect w/Parameters Part 2"
date: 2006-06-04 23:00
comments: true
categories: 
 - Java
 - Programming
 - Struts
 - Web Development
---
In an [earlier post](/blog/2006/01/23/struts-redirect-w-slash-parameters/), I described a class to handle redirects in Struts and passing parameters along. That technique is not necessary; as of Struts 1.2.7, you can use the `ActionRedirect` class instead.

Here's a basic example from inside the execute method in an Action.

{% codeblock lang:java %}
ActionRedirect redirect = new ActionRedirect(mapping.findForward("success"));
redirect.addParameter("myparam","myvalue");
redirect.addParameter("something","fornothing");
redirect.addParameter("answer","42");
return redirect;
{% endcodeblock %}

The `ActionRedirect` class is basically the same as my ForwardParameter class and is probably preferable.