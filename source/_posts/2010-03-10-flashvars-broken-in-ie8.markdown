---
layout: post
title: "FlashVars broken in IE8"
date: 2010-03-10 10:16
comments: true
categories: 
 - Flash
 - Flex
 - HTML
 - Internet Explorer
 - Microsoft
 - Programming
 - Web Development
---
Unsurprisingly, Internet Explorer 8 broke yet another feature of the web.
<!-- more -->
This time, the folks at Redmond broke how Internet Explorer passes the flashvars parameter into Flash. Typically, when placing a Flash object on an HTML page, you use the following syntax:

{% codeblock lang:html %}
<object id="flv" classid="clsid:d27cdb6e-ae6d-11cf-96b8-444553540000" width="692" height="516" codebase="<a href="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=9,0,0,0"">http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#vers...</a> align="middle">
 <param name="allowScriptAccess" value="sameDomain" />
 <param name="allowFullScreen" value="true" />
 <param name="quality" value="high" />
 <param name="bgcolor" value="#424242" />
 <param name="FlashVars" value="name=value" />
 <param name="wmode" value="transparent" />
 <embed type="application/x-shockwave-flash" width="692" height="516" src="<a href="http://www.example.com/mysample.swf"">http://www.example.com/mysample.swf"</a> align="middle" pluginspage="<a href="http://www.macromedia.com/go/getflashplayer"">http://www.macromedia.com/go/getflashplayer"</a> flashvars="name=value" quality="high" bgcolor="#424242" wmode="transparent" allowscriptaccess="sameDomain" allowfullscreen="true" name="flv"></embed>
 <param name="movie" value="<a href="http://www.example.com/mysample.swf"">http://www.example.com/mysample.swf"</a> />
</object>
{% endcodeblock %}

The `OBJECT` tag is used by Internet Explorer and the `EMBED` tag is used by everyone else (Firefox, Safari, etc.). Although the flashvars paramter is not formally described in the [HTML 4.0.1 spec](http://www.w3.org/TR/html4/struct/objects.html#h-13.3.2), this code worked fine with IE 6 and IE 7. Unfortunately, IE 8 does not pass flashvars into the Flash Player. The only work around is to pass the flashvars parameter as part of the movie name parameter, as shown below:

{% codeblock lang:html %}
<object id="flv" classid="clsid:d27cdb6e-ae6d-11cf-96b8-444553540000" width="692" height="516" codebase="<a href="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=9,0,0,0"">http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#vers...</a> align="middle">
 <param name="allowScriptAccess" value="sameDomain" />
 <param name="allowFullScreen" value="true" />
 <param name="quality" value="high" />
 <param name="bgcolor" value="#424242" />
 <param name="FlashVars" value="name=value" />
 <param name="wmode" value="transparent" />
 <embed type="application/x-shockwave-flash" width="692" height="516" src="<a href="http://www.example.com/mysample.swf"">http://www.example.com/mysample.swf"</a> align="middle" pluginspage="<a href="http://www.macromedia.com/go/getflashplayer"">http://www.macromedia.com/go/getflashplayer"</a> flashvars="name=value" quality="high" bgcolor="#424242" wmode="transparent" allowscriptaccess="sameDomain" allowfullscreen="true" name="flv"></embed>
 <param name="movie" value="<a href="http://www.example.com/mysample.swf?name=value"">http://www.example.com/mysample.swf?name=value"</a> />
</object>
{% endcodeblock %}
