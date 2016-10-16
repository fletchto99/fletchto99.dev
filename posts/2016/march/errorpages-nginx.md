title: Simple Custom Nginx Error Pages
disqusIdentifier: 0000000004
keywords:
- nginx
- errors
- customization
- webserver
categories:
- pi
tags:
- nginx
- errors
- customization
- webserver
thumbnailImage: https://images.fletchto99.com/blog/2016/march/errorpages-nginx/thumbnail.jpg
date: 2016/3/15
---

I've noticed that configuring consistent error pages for multiple subdomains/server blocks in Nginx can be quite a pain...
<!-- excerpt -->

{% image fancybox center clear https://images.fletchto99.com/blog/2016/march/errorpages-nginx/banner.jpg %}

### Background

I've noticed that configuring consistent error pages for multiple subdomains/server blocks in Nginx can be quite a pain. I like to have plenty of subdomains setup across multiple server blocks all which have the same error pages. Each time I've got to duplicate the pages to the webroot which has been very inconvenient. I've finally found a way to setup my error pages such that I can simply include a configuration in each of my subdomains and it will automatically redirect errors to the appropriate error page.

This quick tutorial aims to help you set up a way to simply include you existing error pages into existing or new Nginx configurations.

### Prerequisites 

Before beginning create yourself some error pages and store them on your server somewhere. Personally I find it easiest to have them in a consistent location where you won't forget about them for example: `/etc/nginx/errors/errorpages/` but the location is totally up to you. I recommend the dual directory setup as we will be pointing the root for the error pages to the errors folder, while the path will be `/errorpages/` on your domain/subdomains.

### Creating Configuration Files

The configuration file will tell Nginx to look for the error pages along with were the errors should redirect the user to. In my configuration the error pages exist at `sub.domain.com/errorpages/<errorpage>.html` Copy the contents of the configuration file below and save it to `/etc/nginx/errorpage.conf`. It is important to save it in the nginx root directory so that when the configuration file is used later on Nginx will know where to look.

{% codeblock lang:Nginx errorpages.conf %}
error_page 401 /errorpages/401.html;
error_page 404 /errorpages/404.html;
error_page 500 502 503 504 /errorpages/generic.html;

location = /errorpages/401.html {
        root /etc/nginx/errors/;
}

location = /errorpages/404.html {
        root /etc/nginx/errors/;
}

location = /errorpages/generic.html {
        root /etc/nginx/errors/;
}

location = /errorpages/style.css {
        root /etc/nginx/errors/;
}

{% endcodeblock %}

The error_pages directive tells Nginx where to send the user once they've encountered an issue. As you can see it redirects them to `/errorpages/` but as you may know, we don't actually have an `/errorpages/` directory in our webroot. To solve this issue we create specific locations telling Nginx to look in a different directory for errorpages only. We explicitly tell Nginx to look for the required error page files at `/etc/nginx/errors` which we setup earlier.

### Updating Server Configurations

The last step is to update the various server blocks within your `sites-available`. If your server block already contains the `error_page` directives be sure to remove them so that it will use our newly generated configuration. Now all you need to do is `include errorpages.conf` and Nginx will now start to redirect errors to our newly setup error pages. You can include this configuration in any/all of your server blocks and it will adjust accordingly. For example:
 
{% codeblock lang:Nginx my.subdomain.com %}
server {
        listen      80;
        server_name my.subdomain.com;
        
        # Seperate webroot, does not need /errorpages
        root /home/demo/vhost/my.domain.com/;
        
        # Include Nginx errorpages configuration
        include errorpages.conf;

        location / {
                index index.html;
        }
        
}
{% endcodeblock %}

 
 So if you have subdomain a, b and c now a.domain.com/errorpages/, b.domain.com/errorpages and c.domain.com/errorpages will exist.

### Conclusion

Hopefully this has helped you optimize your error page configurations within your Nginx server configuration!

<!-- more -->