title: "Hack Western 3 & CTFort"
disqusIdentifier: 0000000010
keywords:
- ctfort
- hackathons
- nodejs
- angular
categories:
- hackathons
tags:
- ctf
- hackathons
- hacking
thumbnailImage: https://images.fletchto99.com/blog/2016/october/hack-western-3/thumbnail.png
date: 2016/10/16
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

# Friday

After an extremely long bus ride to Western university and arriving around 8:30 pm the team got settled in a room and prepared to start hacking. We spent the first few hours discussing the stack we would use and the API which would ultimately control CTFort. We opted to use Angular 2 with Typescript for the frontend and then Node.js using the express framework on the backend with the data being stored in a postgresql database. All of this was wrapped up and running on an AWS EC2 instance.

Around midnight we finalized our plan and got started with the development. My first job was to get the production environment setup so by the time Sunday morning comes everything can just be uploaded to the server. So my tasks for the night included registering a domain, spinning up an EC2 instance, and setting up the DNS records to point to the AWS instance. I also created the [github repository](https://github.com/fletchto99/CTFort) to host our project.

This was my first time using AWS and I must say I really enjoyed the process. It took only a mere 5 minutes to spin up a Ubuntu 16.04 LTS vm on AWS and get all of our teams public keys added to the instance. I also got the domain name [ctfort.com](https://ctfort.com) registered and then used [Hurricane Electric's free DNS service](https://dns.he.net) to set the A record to point to our server. Next I setup [api.ctfort.com](https://api.ctfort.com) which would be the subomain where our API could be accessed. Next, for fun, I compiled my own version of Nginx with the [more_set_headers](https://github.com/openresty/headers-more-nginx-module#installation) directive so that I could modify there server header, this can also be used as a security measure to change the server tokens preventing malicious crawlers looking for specific server headers. Finally, using my [old guide](https://blog.fletchto99.com/2016/february/letsencrypt-nginx/), I setup Let's encrypt to get free SSL certificates added to our project. Let's encrypt is great because you can have SSL setup in less than 5 minutes.

 That was pretty much all we got done friday night, we ended up going to sleep around 4:30AM.

 # Saturday

The goal for saturday was to finish up the project. This included creating the entire UI and creating all of the API endpoints, so there was plenty to get done. After waking up at 8:30 the team got started right away. Kurt & Ryan started by creating the UI landing page (with an awesome parallax effect) & login/registration forms. My job was to create the backend API, build the database schema and setup PostgreSQL on the server.

After creating the schema I got PostgreSQL installed on the server. To use this I just used apt to install the latest version of postgres. I then proceeded to create a new user on the box with limited permissions on the fs and the databases. This user would be used to connect to the database and also run the web application/API. I did this so that if an attacker were able to gain access to this user they would be restricted to the permissions of that account only.

The last step was to build the RESTful API. I decided to use Node.js + Express to build the API since I'm pretty familiar with them. Most of the API was straight forward, I had the endpoints in `app/controllers/`, the endpoints manipulated the models at `app/models/` and the models would access the database. The `app/shared/` directory contained a bunch of helper functions used throughout the application.

I also had a layer of middleware in `app/middleware/` which performed various operations. The [helmet](https://github.com/helmetjs/helmet) middleware was used as a general security filter to prevent things like XSS and click jacking. I used express-session with the pg storage model to store sessions securely within the database. Finally I created an auth middleware which ensures all access to `<website>/auth/` is using a validated session. To store the passwords in the database I chose to use the industry standard [bcrypt library](https://www.npmjs.com/package/bcrypt) which quickly and securely stores passwords.

Building the API took the majority of the day and the rest of the day was spend at

# Saturday Events

https://github.com/okabe/hackwestern
<!-- more -->