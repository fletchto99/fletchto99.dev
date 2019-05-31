title: "Minecraft mods: reversing style"
keywords:
- security
- gaming
- minecraft
tags:
- security
- gaming
- minecraft
categories:
- technical
thumbnailImage: https://images.fletchto99.com/blog/2019/may/minecraft-mod-reversing/thumbnail.jpg
date: 2019/05/13
---

This past weekend a co-worker bought his son a Minecraft mod and unfortunately, even after payment, it still refused to allow them to use it! This was nothing a bit of simple reverse engineering couldn't solve.

<!-- excerpt -->

{% image center clear https://images.fletchto99.com/blog/2019/may/minecraft-mod-reversing/banner.png %}

This past weekend a co-worker bought his son a Minecraft mod and unfortunately, even after payment, it still refused to allow them to use it! This was nothing a bit of simple reverse engineering couldn't solve. Thankfully, my co-worker came up with [this nifty solution](https://carnal0wnage.attackresearch.com/2019/05/minecraft-mod-mothers-day-and-hacker-dad.html) to get the mod working after they had purchased it. While his solution was great for a quick fix we discussed the possibility of moving it from a heavy 3rd party approach to a simpler approach using code.

## Research

Before jumping into coding it's always important to perform a bit of research to determine the best approach, especially when it comes to reverse engineering. So I popped open JD-GUI (it's been too long) and opened up the files of interest which my colleague had pointed:

{% image fancybox center clear https://images.fletchto99.com/blog/2019/may/minecraft-mod-reversing/1.png "The function performing the authentication check" %}

My first idea was to simply inject some code which preventing the `betaTesters(...)` event handler from ever being executed. Unfortunately, after some quick research, I determined this approach required using bytecode manipulation libraries such as [ASM](https://asm.ow2.io/) or [BCEL](https://commons.apache.org/proper/commons-bcel/) which isn't exactly lightweight. Realizing that injection like this would no longer work it was back to the drawing board.

Based off of my colleague's research, it looks like the mod checks against a pre-defined list of minecraft user UUIDs which is loaded from pastebin. Instead of forcing the authentication to exit early, what if we could somehow inject our name into that list? Surely that would work as well.

{% image fancybox center clear https://images.fletchto99.com/blog/2019/may/minecraft-mod-reversing/2.png "The code which loads the list of allowed users" %}

After some more spelunking in JD-GUI it looks like the author left us a class variable to manipulate. Inside of the script which loads the authorized users is a _private_ class variable `testers` which is populated during runtime from the pastebin link. _Sidenote: this means paid user's can't use it offline! What?!?!_ However, if we could add our name to that arraylist then we're golden!

Java offers up a handy set of [Reflection APIs](https://docs.oracle.com/javase/tutorial/reflect/) which are often used to observe the runtime of an application. Reflection will actually allow us to do a little more than just observe in this case but before I go into too much detail there's a few things I needed to check to make sure this approach would work:

1. Is there a Java security manager in place to prevent the use of reflection?
2. Do we have access to the classloader which loaded the original mod (I.E. we're not sandboxed)?
3. When does the authentication check for the mod occur? This will affect how we load & inject.
4. Can we guarantee that the mod is loaded before our injection such that we'll have access to `testers` during runtime?

After a bit of research I came up with these answers:

1. No security manager was in place... I was able to perform a simple reflection test within my mod.
2. [Forge](https://mcforge.readthedocs.io/en/latest/), the minecraft mod loader, loads mods during runtime. It's likely all mods are loaded via the same classloader and there is no sandboxing. A simple test for this was to see if I had access to classes outside of my mod during runtime.
3. The authentication check occurs when the user attempts to load a world, so we have plenty of time to ensure our injection occurs meaning we can load our injection once the mod is in memory. This makes accessing the original mod's classes much easier since there's no racing.
4. Forge offer us the ability to set dependencies so we can guarantee their mod loads before our injection mod.

So now all we needed to do was get our [Minecraft user's UUID](https://mcuuid.net/) injected into the `testers` variable during runtime and we're off to the races.

## Building the Injection Mod

Oh Java... how I haven't missed you. After roughly an hour of dependency hell and debugging my java environment I _finally_ managed to get minecraft and forge loading to the same versions that my colleague's son plays on. Once I got all of that sorted out I made a simple "hello world" mod which loaded after the original mod. Based on the logs I could see the load order was being followed and all that was left was to inject our user into this `testers` variable.

Using the reflection APIs I was able to come up with this first pass hard coded UUID test:

{% codeblock lang:Java ModLoader.java %}
  // The class, including the package, which the testers variable exists in
  String clazz = "the.mod.Class"
  String uuid = "my-minecraft-uuid"

  // Inject into the authorized users variable.
  Field testers = Class.forName(clazz).getDeclaredField("testers");
  testers.setAccessible(true);
  ((ArrayList<String>)testers.get(null)).add(uuid);
{% endcodeblock %}

Breaking this down the `Class.forName(clazz)` will find the mod's checker class during runtime and `getDeclaredField("testers")` will allow us to access to the testers field. At this point we can't do anything with that field other than observe. Thankfully the Reflection API provides us a [setAccessible(...)](https://docs.oracle.com/javase/7/docs/api/java/lang/reflect/AccessibleObject.html#setAccessible(boolean)) which we can use to make this field accessible from within our class, even though the original class set it to private. Lastly, all we needed to do was to inject our UUID into the the `testers` ArrayList. Since the author set it as a static variable we're able to retrieve the field's value directly using [get(null)](https://docs.oracle.com/javase/7/docs/api/java/lang/reflect/Field.html#get(java.lang.Object)). Since we know testers is of type `ArrayList<String>` we can cast to that and simply call `.add(...)` with our UUID.

Unfortunately things didn't quite work as expected... I was still locked out! I wonder why? After some more spelunking in JD-GUI it turns out that this check occurs multiple times throughout the mod. Thankfully the author appears to have copy/pasted the code so we were able to re-use the injection process giving us:

{% codeblock lang:Java ModLoader.java %}
  // The classes we need to inject into
  String[] clazzes = {
    "the.mod.Class1",
    "the.mod.Class2"
  };

  String uuid = "my-minecraft-uuid";

  for (String clazz : clazzes) {
    Field testers = Class.forName(clazz).getDeclaredField("testers");
    testers.setAccessible(true);
    ((ArrayList<String>)testers.get(null)).add(uuid);
  }
{% endcodeblock %}

**Boom! We're in!** The last problem was allowing this to work for my colleague and his son. A hardcoded UUID wouldn't work well here. My initial approach at solving this hurdle was to parse the user's UUID from the web but once again we faced the same problem as the mod: no offline access! What's my colleague's son going to do when the internet goes out? Surely there was a better way to get this information without needing online access. Thankfully with the help of google and IntelliJ I was able to piece together a way to parse the user's UUID without needing internet:

`Minecraft.getMinecraft().getSession().func_148256_e().getId().toString()`

A bit of explination how that did the magic I needed: Through googling I found that the type `net.minecraft.util.com.mojang.authlib.GameProfile` had a getter for the user's UUID. Unfortunately this wasn't simply accessible via forge, since the minecraft client uses some extremely basic obfuscation techniques. However with IntelliJ I was able to find that the fancy `func_148256_e()` which was typehinted to be the `GameProfile` instance I was looking for.

And putting it all together we end up with:

{% codeblock lang:Java ModLoader.java %}
  // The classes we need to inject into
  String[] clazzes = {
    "the.mod.Class1",
    "the.mod.Class2"
  };

  // Retrieve the UUID
  String uuid = Minecraft.getMinecraft().getSession().func_148256_e().getId().toString();

  for (String clazz : clazzes) {

    // Load the class
    Field testers = Class.forName(clazz).getDeclaredField("testers");

    //Make it accessible (I.E.) not private
    testers.setAccessible(true);

    //Add our UUID
    ((ArrayList<String>)testers.get(null)).add(uuid);
  }
{% endcodeblock %}

Of course all of that was wrapped in some boilerplate mod code. Furthermore, the injection mod's manifest allowed me to specify that it depended on the other mod being loaded, which gives us 2 wins: we guarantee the other mod loads first and we guarantee the classes we're looking for will exist at runtime. I didn't really do any error handling other than just ignoring all errors so minecraft wouldn't crash.

It's been a while since I've touched minecraft but this made for a fun evening. Remember if you enjoy using these mods make sure to support the authors! I used to [make plugins](https://dev.bukkit.org/members/fletch_to_99/projects) for minecraft and it really meant a lot to me when someone paid/donated! Hopefully you learned something and if you're planning on using any knowledge you gained here, please make sure to use it for good. Support indie developers :)


<!-- more -->
