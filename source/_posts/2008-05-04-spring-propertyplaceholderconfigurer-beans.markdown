---
layout: post
title: "Spring PropertyPlaceholderConfigurer beans"
date: 2008-05-04 23:00
comments: true
categories: 
 - Programming
 - Java
 - Spring
---
So I found a scenario today where we had two different `PropertyPlaceHolderConfigurer` beans declared in two separate XML files. Unfortunately, the second property file wasn’t getting loaded, so we would get errors about Spring being unable to resolve particular properties.

I found this great post summarizing the solution:

[http://dotal.wordpress.com/2007/09/14/mulitple-propertyplaceholderconfigurer-configurations/](http://dotal.wordpress.com/2007/09/14/mulitple-propertyplaceholderconfigurer-configurations/)

Basically, you have to update the definition for the first `PropertyPlaceHolderConfigurer` bean that gets loaded and add the `ignoreUnresolvablePlaceholders` property:

{% codeblock lang:xml %}
<bean id="mailProps" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
         <property name="location" value="classpath:mail.properties"/>
         <property name="ignoreUnresolvablePlaceholders" value="true"/>
</bean>	
{% endcodeblock %}