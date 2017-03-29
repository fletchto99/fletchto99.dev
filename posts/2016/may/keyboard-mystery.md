title: The Shocking Keyboard Mystery
disqusIdentifier: 0000000006
keywords:
- mystery
- keyboard
- ottawa
categories:
- personal
tags:
- corsair
- keyboard
- hardware
thumbnailImage: https://images.fletchto99.com/blog/2016/may/keyboard-mystery/thumbnail.png
date: 2016/5/1
---

For the last month or so my aluminum body keyboard has been shocking me for intervals of about 10 seconds. I've finally figured out why...
<!-- excerpt -->

{% image fancybox center clear https://images.fletchto99.com/blog/2016/may/keyboard-mystery/banner.png %}

For the last month or so my aluminum body keyboard has been shocking me for intervals of about 10 seconds. I've finally figured out why this has been occurring. I was going crazy when my keyboard was continuously shocking me while my room mate, who right after me touched the keyboard and it didn't zap him at all. Turns out I'm not going crazy *yet*.

About a month ago I got a new desk and a new computer case. About a week after getting them the random zappage started to occur. I thought there's no way it has anything to do with my desk, so off to Corsair support I go for my case & keyboard. Just for safety measures I also poked EVGA support to see if it might be a PSU issue. Finally for good measure I posted on [LinusTechTips](https://linustechtips.com/main/topic/585942-my-keyboard-zapped-me-is-this-normal/) too see if any community member had experienced this before.

It took me about a month of talking to support forums and on LTT to get nowhere; the case, PSU & keyboard all checked out. Everything was grounded fine and working as intended. I was stumped... that is until the other day when I was gaming and getting shocked quite a bit. When I'm gaming I usually like to rest my feet on the subwoofer below my desk as seen in the image below. Oh, and yes that is a 220V old dryer plug below my desk, harmless right? WRONG. In a normal situation this would be fine, since I'm not plugging my toes in or anything... But this situation isn't normal.

{% image fancybox center clear https://images.fletchto99.com/blog/2016/may/keyboard-mystery/1.jpg "Keyboard Relative to Subwoofer" %}


So out came the multi-meter as seen in the images below. One end touching some *exposed* metal on the plug which should be grounded, the other in the ground of a known to be working plug. Sure enough the results were quite interesting. Turns out the plug has shorted and has been using my body to complete the circuit and use my computer as a ground. I had 65V going through me and into the keyboard, it was quite shocking to find out this was the cause!

{% image fancybox fig-50 left https://images.fletchto99.com/blog/2016/may/keyboard-mystery/2.jpg "Probing the Exposed Metal" %} {% image fancybox fig-50 right https://images.fletchto99.com/blog/2016/may/keyboard-mystery/3.jpg "Reading on the Multi Meter" %}

Thankfully this mystery has been solved and no equipment was damaged or injuries occurred in the process. I have since contacted my landlord and will have this issue resolved ASAP.

<!-- more -->
