title: ASUS broken API authentication
keywords:
- asus
- whitehat
- responsible disclosure
- security disclosure
- vulnerability
- broken authentication
categories:
- security disclosures
tags:
- responsible disclosure
- asus
- whitehat
thumbnailImage: https://images.fletchto99.com/blog/2016/september/asus-disclosure/thumbnail.png
date: 2016/9/5
---

About 3 months ago a component on my motherboard broke, so off I went to contact ASUS for an RMA only to find something completely unexpected...
<!-- excerpt -->

{% image center clear https://images.fletchto99.com/blog/2016/september/asus-disclosure/banner.png %}

So this is my first public disclosure... I can't believe I found this.

About 3 months ago a component on my motherboard broke, so off I went to contact ASUS for an RMA only to find out that their RMA authentication mechanism was broken, badly. It all started after a google search for ASUS' RMA page which brought me to the typical form asking for the usual RMA information including your name, product and serial numbers. Lastly your need to fill out a verification code to prevent spambots. After completely filling out this form I pressed the submit button.... and nothing happens. So I scroll up & down to confirming that all fields were filled in properly. After verifying the integrity of the data I had entered I press submit again and once again nothing happens.

So off to the network inspector I go in chrome. It appears that the request is failing with a 500 response code. So as a developer my instinct was to see what ASUS does to handle invalid responses, do they just ignore them and not notify the user? So I hop on over to the sources tab only to be completely shocked about what I was going to see!

{% image fancybox center clear https://images.fletchto99.com/blog/2016/september/asus-disclosure/1.png "AJAX code snippet... usernameAndPassword?" %}

After inspecting the code I was most certainly not logged in, so there's no way ASUS can be serving me up a custom page with my credentials hardcoded into the page. Now perhaps this was just an oversight on my part and there is a comment above the code, maybe it clarifies the use of this `usernameAndPassword` variable. So off to google translate I go.

{% image fancybox center clear https://images.fletchto99.com/blog/2016/september/asus-disclosure/2.png "Translation of comment" %}

Nope, not really useful. So far we know that there is a `usernameAndPassword` hardcoded value which is being set as an authorization header for an API call. The variable was in the format of `var usernameAndPassword = 'Authorization: Basic c29tZVVzZXI6c29tZVBhc3M='` (note the value has been changed) and this is being posted to all API endpoints. After some basic analysis and understanding how basic authentication works we can determine that the value is actually just the base64 encoded credentials for whatever account this is. This data is **NOT** encrypted nor hashed and can easily be reversed by [decoding the base64](http://base64decode.net/) string.

{% image fancybox center clear https://images.fletchto99.com/blog/2016/september/asus-disclosure/3.png "Decoding base64 encoded data" %}

 So surly this is just some low level account that *isn't* really necessary and doesn't have any access... fingers crossed. So after some simple research I was able to find the pretty login page to the RMA system by going to the root of the API. No complex URL fuzzing required, it was found in about 30 seconds. *Side note: Shouldn't this kind of login page be only accessible internally?*

{% image fancybox center clear https://images.fletchto99.com/blog/2016/september/asus-disclosure/4.png "Pretty login page for ASUS RMA system" %}

 So far we have some login credentials and a login page. The last step is to actually check if the page accepts the credentials and where it brings you. And tada I'm in. So at this point I'm freaking out panicking to find the logout button and get this reported ASAP. From my brief 5 seconds in the system it appears that the account was not only valid but also an administrator account.

{% image fancybox center clear https://images.fletchto99.com/blog/2016/september/asus-disclosure/5.png "Holy crap, we're in boys!" %}

Ok so now that I've found this I needed to report it. Before I send it off I do a little bit more information gathering determining that there are a total of 3 forms which have this information hardcoded within. After gathering all of the above information and taking the relevant screenshots I put together a report to send ASUS... only where do I send it to? ASUS doesn't have a fancy whitehat program like most tech giants so I had to search for a secure way to contact them. After a while of looking I came across one line in their privacy policy stating if a technical vulnerability is found I should email [privacy@asus.com](mailto:privacy@asus.com). So I sent them a report with all of the required information.

{% image fancybox center clear https://images.fletchto99.com/blog/2016/september/asus-disclosure/6.png "Privacy policy statement - No whitehat" %}

On July 13th I sent the email off to ASUS containing the required information. On the 14th I noticed that they have removed the affected lines of code on 2/3 forms. Later that night they reply in a one line email saying thank you and that the issue has been resolved. I reply to ASUS notifying them that only 2 of the 3 affected URLs have been fixed. After a week of hearing nothing back I notice that the final form has been silently patched. A few days later I send a follow up email asking if everything should have been patched but I have not heard back from ASUS since then. It has been over 45 days and according to [CERT](http://www.cert.org/vulnerability-analysis/vul-disclosure.cfm) that should be plenty of time to fully fix the issue and therefore I am able to publicly disclose the issue.

## Official Timeline

```
July 13, 2:00  PM EST - Their livechat directes me to an RMA form, telling me it will likly cost $ to fix my product
July 13, 2:15  PM EST - Exploit found
July 13, 2:30  PM EST - Exploit reported to privacy@asus.com
July 14, 1:20  PM EST - Exploit patched on the RMA forms (--redacted--) urls but not on the esupport form (--redacted--) url
July 14, 11:23 PM EST - Explot "fixed". There is still an affected URL (--redacted--)
July 15, 10:30 AM EST - I notify ASUS of this other page which is still affected
July 21, 2:00  PM EST - Looks like the second issue is fixed, haven't heard back from ASUS though
July 25, 4:50  PM EST - I send a followup with ASUS asking the status of the issue & recommending them to have a better form of contact for vulnerabilities
September 5, 8:30 PM EST - Publically disclose the issue
September 22, 2:45 AM EST - Asus follows up saying thank you for the report and all issues have been fixed
```


Oh and in the end of all of this they still wanted over $250 to fix my motherboard. So I ended up fixing it my self by ordering a replacement cable for 0.99 on ebay.
<!-- more -->
