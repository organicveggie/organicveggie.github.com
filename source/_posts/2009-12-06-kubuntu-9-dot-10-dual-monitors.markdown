---
layout: post
title: "Kubuntu 9.10 Dual Monitors"
date: 2009-12-06 11:12
comments: true
categories: 
 - KDE
 - Kubuntu
 - Linux
 - Ubuntu
 - Xorg
 - xrandr
---
A few months back I got a new [12.1" System76 Darter laptop](http://system76.com/product_info.php?cPath=28&products_id=76) and I finally decided to setup dual monitors on Kubuntu 9.10 (Karmic Koala).What I wanted was to have my laptop LCD on the left at 1280x800 and my ViewSonic LCD panel on the right running at 1680x1050. Apparently I'm old-skool, because I'm used to hacking away at my `xorg.conf` just to get dual monitor support to work under Linux. Somehow I never knew about [xrandr](http://www.x.org/wiki/Projects/XRandR), KRandR, etc.
<!-- more -->
[ThinkWiki](http://www.thinkwiki.org/wiki/ThinkWiki) has a great [article about using XRandR][ThinkWikiArticle] that really helped me out. When I first launched Kubuntu the two screens were mirrored. So, I tried to use KRandR to set the screens side by side. Unfortunately, it seemed to be confused - it saw there were two outputs, but it thought they were the same screen. This is where the ThinkWiki article came in handy.

With the later versions of Xorg, Ubuntu doesn't even include an `xorg.conf`. Instead of even monkeying around with Xorg, I just used xrandr. There were three key steps:

## Identify Outputs

{% codeblock lang:bash %}
$ xrandr -q
Screen 0: minimum 320 x 200, current 2960 x 1050, maximum 8192 x 8192
VGA1 connected 1680x1050+1280+0 (normal left inverted right x axis y axis) 474mm x 296mm
   1680x1050      60.0*+
   1280x1024      75.0
   1152x864       75.0
   1024x768       75.1     70.1     60.0
   832x624        74.6
   800x600        72.2     75.0     60.3     56.2
   640x480        72.8     75.0     66.7     60.0
   720x400        70.1
LVDS1 connected 1280x800+0+0 (normal left inverted right x axis y axis) 261mm x 163mm
   1280x800       60.0*+
   1024x768       60.0
   800x600        60.3
   640x480        59.9
DP1 disconnected (normal left inverted right x axis y axis)
{% endcodeblock %}

This shows I have two main outputs connected right now: VGA1 and LVDS1. Under Ubuntu 9.04 (Jaunty), my outputs were labeled VGA and LVDS.

## Disable secondary output

Once we know the name of the outputs, we can disable the secondary output. This is the key step to getting a large virtual desktop working without something like Xinerama. If you don't disable the secondary output, Xorg never seems to be able to successfully distinguish between the two outputs.

{% codeblock lang:bash %}
$ xrandr --output VGA1 --off
{% endcodeblock %}

## Re-enable both outputs

When we re-enable the outputs we can specify the location of the secondary display, relative to the primary. We can also let xrandr figure out the best resolution for each:

{% codeblock lang:bash %}
$ xrandr --output LVDS --auto --output VGA --auto --right-of LVDS
{% endcodeblock %}

Voila. Finally, if you want to automate this, the [ThinkWiki article][ThinkWikiArticle] has a great little script you can use. However, I did have to modify it slightly ... I had to force the VGA1 output off before setting them both to auto. Without that, the secondary screen remained blank.

{% codeblock /etc/X11/Xsession.d/45custom_xrandr-settings lang:bash %}
# If an external monitor is connected, place it with xrandr
 
# External output may be "VGA" or "VGA-0" or "DVI-0" or "TMDS-1"
EXTERNAL_OUTPUT="VGA1"
INTERNAL_OUTPUT="LVDS1"
# EXTERNAL_LOCATION may be one of: left, right, above, or below
EXTERNAL_LOCATION="right"
 
case "$EXTERNAL_LOCATION" in
       left|LEFT)
               EXTERNAL_LOCATION="--left-of $INTERNAL_OUTPUT"
               ;;
       right|RIGHT)
               EXTERNAL_LOCATION="--right-of $INTERNAL_OUTPUT"
               ;;
       top|TOP|above|ABOVE)
               EXTERNAL_LOCATION="--above $INTERNAL_OUTPUT"
               ;;
       bottom|BOTTOM|below|BELOW)
               EXTERNAL_LOCATION="--below $INTERNAL_OUTPUT"
               ;;
       *)
               EXTERNAL_LOCATION="--left-of $INTERNAL_OUTPUT"
               ;;
esac
 
xrandr |grep $EXTERNAL_OUTPUT | grep " connected "
if [ $? -eq 0 ]; then
    xrandr --output $EXTERNAL_OUTPUT --off
    xrandr --output $INTERNAL_OUTPUT --auto --output $EXTERNAL_OUTPUT --auto $EXTERNAL_LOCATION
    # Alternative command in case of trouble:
    # (sleep 2; xrandr --output $INTERNAL_OUTPUT --auto --output $EXTERNAL_OUTPUT --auto $EXTERNAL_LOCATION) &amp;
else
    xrandr --output $INTERNAL_OUTPUT --auto --output $EXTERNAL_OUTPUT --off
fi
{% endcodeblock %}

Place this in `/etc/X11/Xsession.d` as `45custom_xrandr-settings` and it will automatically run.

[ThinkWikiArticle]: http://www.thinkwiki.org/wiki/Xorg_RandR_1.2