title: "Hack Western 3 & CTFort"
keywords:
- ctfort
- hackathons
- nodejs
- angular
categories:
- events
tags:
- ctf
- hackathons
- hacking
thumbnailImage: https://images.fletchto99.com/blog/2016/october/hack-western-3/thumbnail.png
date: 2016/10/23
---

This past weekend I had the opportunity to participate in Hack Western 3 and while I didn't win I still learned alot!
<!-- excerpt -->

{% image center clear https://images.fletchto99.com/blog/2016/october/hack-western-3/banner.png %}


This past weekend I had the opportunity to participate in Hack Western 3 which was a 36 hour hackathon located at Western University! In case you're unfamiliar with the concept of a hackathon [MLH](https://mlh.io/faq#what-is-a-hackathon) states that it is:

{% blockquote %}
A hackathon is best described as an “invention marathon”. Anyone who has an interest in technology attends a hackathon to learn, build & share their creations over the course of a weekend in a relaxed and welcoming atmosphere. You don’t have to be a programmer and you certainly don’t have to be majoring in Computer Science.
{% endblockquote %}

# Team & Idea Formation

Both my roommate and I were accepted to Hack Western 3, so we opted to work as a team. On the 9 hours bus ride from Ottawa to Western we met Ryan, our third teammate. Over the past few months I've been thinking about building a centralized system to aid hackers in CTFs. The idea comes from previous CTF events I've attended where there is no convenient way manage which flags have been captured and how they were captured. CTFort aims to ease the pain of managing which flags have been captured and will hopefully some day act as a centralized portal for CTF teams.

{% image fancybox center clear https://images.fletchto99.com/blog/2016/october/hack-western-3/1.png "CTFort Logo" %}

# Friday

After an extremely long bus ride to Western university and arriving around 8:30 pm the team got settled in a room and prepared to start hacking. We spent the first few hours discussing the stack we would use and the API which would ultimately control CTFort. We opted to use Angular 2 with Typescript for the frontend and then Node.js using the express framework on the backend with the data being stored in a postgresql database. All of this was wrapped up and running on an AWS EC2 instance.

Around midnight we finalized our plan and got started with the development. My first job was to get the production environment setup so by the time Sunday morning comes everything can just be uploaded to the server. So my tasks for the night included registering a domain name, spinning up an EC2 instance, and setting up the DNS records to point to the AWS instance. I also created the [github repository](https://github.com/fletchto99/CTFort) to host our project.

This was my first time using AWS and I must say I really enjoyed the process. It took only a mere 5 minutes to get registered and spin up a Ubuntu 16.04 LTS vm on AWS and get all of our teams public keys added to the instance. I also purchased the domain name [ctfort.com](https://ctfort.com) registered and then used [Hurricane Electric's free DNS service](https://dns.he.net) to set the A record to point to our server. Next I setup [api.ctfort.com](https://api.ctfort.com) which would be the subomain where our API could be accessed. Next, for fun, I compiled my own version of Nginx with the [more_set_headers](https://github.com/openresty/headers-more-nginx-module#installation) directive so that I could modify there server header, this can also be used as a security measure to change the server tokens preventing malicious crawlers looking for specific server headers. Finally, using my [old guide](https://blog.fletchto99.com/2016/february/letsencrypt-nginx/), I setup Let's encrypt to get free SSL certificates added to our project. Let's encrypt is great because you can have SSL setup in less than 5 minutes.

That was pretty much all we got done friday night, we ended up going to sleep around 4:30AM after submitting our project to the [Hack Western 3 devpost](https://devpost.com/software/ctfort).

# Saturday

The goal for saturday was to finish up the project. This included creating the entire UI and creating all of the API endpoints, so there was plenty to get done. After waking up at 8:30 the team got started right away. Kurt & Ryan started by creating the UI landing page (with an awesome parallax effect) & login/registration forms. My job was to create the backend API, build the database schema, and setup PostgreSQL on the server.

After creating the schema I got PostgreSQL installed on the server. To do this I just used `apt-get` to install the latest version of postgres. I then proceeded to create a new user on the box with limited permissions on the fs and the databases. This user would be used to connect to the database and also run the web application/API. I did this so that if an attacker were able to gain access to this user they would be restricted to the permissions of that account only.

The last step was to build the RESTful API. I decided to use Node.js + Express to build the API since I'm pretty familiar with them. Most of the API was straight forward, I had the endpoints in `app/controllers/`, the endpoints manipulated the models at `app/models/` and the models would access the database. The `app/shared/` directory contained a bunch of helper functions used throughout the application.

I also had a layer of middleware in `app/middleware/` which performed various operations. The [helmet](https://github.com/helmetjs/helmet) middleware was used as a general security filter to prevent things like XSS and click jacking. I used express-session with the pg storage model to store sessions securely within the database. Finally I created an auth middleware which ensures all access to `<website>/auth/` is using a validated session. To store the passwords in the database I chose to use the industry standard [bcrypt library](https://www.npmjs.com/package/bcrypt) which quickly and securely stores passwords.

Building the API took the majority of the day and the rest of the day was spent at various events. We ended up submitting our project sunday at 4:30 in the morning so we could catch a good night's sleep. Unfortunately we didn't get everything done but thankfully we had enough to pitch our idea!

{% image fancybox center clear https://images.fletchto99.com/blog/2016/october/hack-western-3/2.jpg "*cough* Assassin's creed unity *cough*" %}

# Saturday Events & 4th Place in Forensics Challenge

There were 2 events which I chose to attend on saturday, both of which were done by security related companies. The first was a talk by an employee from Digital Boundary Group titled "Internet Cartography: Mapping the Internet" and the second was "Cracking the Code: How to use Decryption to Uncover the Truth" by Magnet Forensics.

The talk by Digital Boundary Group can be found here: [https://github.com/okabe/hackwestern](https://github.com/okabe/hackwestern). It was a great talk! The TL;DR version is essentially one of the Digital Boundary Group employees mapped the internet & its IRC servers. He found many interesting things including IRC Botnets, Torrent Release Groups and even some local London, Ontario IRC server! He did this by using [Hurricane Electric's BGP services](http://bgp.he.net/) to find top level IPs for specific, countries and ISPs and then mapping out their networks and finally looking for servers which responded with IRC like traffic on port 6667. I highly suggest taking a look at the presentation for an in depth explanation.

The second event was a forensics challenge provided Magnetic Forensics to demonstrate how forensics can be very difficult. I ended up placing **fourth** in this challenge! For this challenge we were given a phone backup of a person who had committed a theft. Our objectives were to find the apps he used to communicate with the other thieves, the thieves online names, the location history of the phone, and if possible the real name of the owner of the phone. What made this challenge complex was that there were hundreds of apps and thousands of files/folders.

For this challenge we were given a few hints saying look for: odd files in the downloads folder, log files, chat database, .eml files and app (Pokemon GO) that might track location data. After a recursive find I came across all of the *.eml files, which had revealed that there were 3 users involved: A, B and C. I also looked for the log files which were from an IRC app and they too had the same 3 users. Also in the search for log files were the pokemon go logs for the location history. Next I found a sql lite database for Whatsapp which had the same 3 usernames but their passwords were encrypted. When browsing the downloads folder I noticed there was a key.store file, so using this key file I was able to decrypt the realnames which were encrypted with AES-128. Finally to determine the real name of phone's owner I noticed that the owner joined IRC as user "A" which matched up with the person's decrypted realname in the whatsapp database.

# Closing Ceremonies

Late sunday morning we pitched our idea to many judges. Many of them seemed quite impressed with the idea and were really interested in what we learned. One of the judges though that it was awesome that we compiled our own version of Nginx to change the server header. He also seemed quite intrigued about the fact that we supported SSL through Let's Encrypt. Also the group beside us made a game for the Rift so I got to try VR for the first time! Unfortunately we didn't win this time around but it was a great event! I plan to continue building out CTFort and if you're interested I encourage you to also contribute: [https://github.com/fletchto99/ctfort](https://github.com/fletchto99/ctfort).

{% image fancybox center clear https://images.fletchto99.com/blog/2016/october/hack-western-3/3.jpg "Closing Ceremonies" %}

<!-- more -->
