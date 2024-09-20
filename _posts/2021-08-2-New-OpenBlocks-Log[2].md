---
title: New OpenBlocks Log[2]
author: NurIhsan
date: 2021-08-08 19:13:00 +0700
categories: Projects New-OpenBlocks
tags: project kotlin android new-openblocks status
---

Willkommen to The New OpenBlocks log index 2!

Today, is a BIG day! well, not today, but at the time I discovered _something you might be curious about_.

Anyway, I took a bit of a break for a few days writing these blogs because I was quite focused on the project (and also because I need to attend to my online school again, my free time is reduced significantly :( )

I'm a bit sorry because I didn't update you guys right away when I discovered _something you might be curious about_, okay okay, let's get to the point.

# Libraries
As I've said in the previous log, I was planning to create a some kind of library mechanism so people could add libraries to reduce writing stuff from scratch and also be lazy like me. If you don't know, library is basically a piece of code that can be bundled with your app to make it's developer's life much easier, It's like having a helper. Libraries often includes functions whose other people might not be able to create or implement their own, so they chose to implement this library inside their app to use those features without knowing how they work (and maintaining them), simply a black box.

<img src="/assets/img/library-diagram.svg" style="background: white;" alt="Library diagram">

<p style="font-size: 12px;" align="center">Diagram of how a library is being used. By Kővágó Zoltán (DirtY iCE) - self-made, based on file:Libs_dia.png, CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=2985841</p>

Android libraries are usually packaged in the `.aar` format (Android Archive), they can contain compiled java codes, resources, manifest etc, that are usually used in an app (but can be bundled into an app).

Since an `.aar` file is just a zip file, we can use `ZipInputStream` to extract files in it, but wait, what is the standard? what file structure is used in this aar file? which files are which? for that, you can take a look at it's anatomy [here](https://developer.android.com/studio/projects/android-library.html#aar-contents). Since we wanted a simple, proof-of-concept library implementation, we're just going to take it's resources (`/res`) and it's jarfile (`/classes.jar`).

![What we're going to use](/assets/img/aar-what-were-going-to-use.png)

But wait, the jarfile `classes.jar` is in java bytecode! "why is it in java bytecode when we need a dalvik one to be able to add it within the app?" It's because we need the java bytecode for the compiler to recognize which function / field / class is used by a java code and since it's a java bytecode, we can dex it into dexfile when we needed it (we can't transform back a dalvik bytecode into java bytecode).

It seems like we need an extra processing to be able to use this library, and since transforming java bytecode into dalvik (and compiling it's resources using aapt2) takes quite a lot of resources, why not let the user to be able to compile and cache the library as they please? Yeah, caching the library.

Basically, the user can compile the library into a cache when they wanted to, and once we need it for compiling an app, it will use that cache instead of compiling it from scratch, aaand for extra laziness, we should also allow the user to import a precompiled library so they don't need to compile it when they need to use it! amazing!

So I followed what I wanted to do and it ended up working perfectly, libraries like `androidx.core` gets compiled into `res.zip` and `classes.jar` that I would be able to use it in my app!

But, there is a very weird bug, for no apparent reason, `aapt2` doesn't like to compile big libraries, like appcompat. When I tried compiling appcompat, and on the resource compilation stage, `aapt2` seem to just give up after writing few kilobytes of compiled resources, how do I know that? my CPU usage is very low compared to few seconds after `aapt2` tries to compile the library, I have no idea why this issue is happening, it might be a bug in `aapt2` itself, or maybe a bug in the port, since compiling these libraries work perfectly in PC.

If you know out what's up, you can head on to this [issue](https://github.com) and hit me up, we need some explanations! Thanks! (I'll change this to the real issue once the new openblocks got released).

Okay, let's just slide that issue away and forget about it. yay, we got the library system working!

# Modules
Now, for the most exciting part, **modules**. Modular is the SPECIAL part of openblocks, no other app, and I mean NO OTHER APP has EVER thought of this idea. Most apps are hardcoded to it's core, everything is non-changeable and the user just has to depend on the developer of the app to have stuff you want to be implemented in the app.

## Story of failure

<img align="left" src="/assets/img/stupid-openblocks-bug.png" alt="the stupid openblocks bug" height="300" style="margin-right: 16px"/>

People might've been asking is "modules" is the part that we got into trouble with on developing openblocks? I'd say kinda, we (to that I mean ME) got into trouble with the module storage system, which might sound stupid, but back then I was very stupid in developing OpenBlocks. On the openblocks time, I was very stupid, I didn't test my codes, not a single one, all I did was just leave that critical error alone and go ahead develop other stuff. But little did I know, that critical error is very annoying to find + since I can't test other stuff that I've spent my time on instead of debugging, there must be bugs there! aaa.

Now, I've learnt my lesson, I ALWAYS need to test my code everytime I finished working on it, and also, since the new openblocks is going to be written in kotlin, it makes developing much more easier and less-buggy!

I can't believe I said it, programming in kotlin makes less bugs than programming in java. Even though I agree that kotlin is a bit more bloated than java, it really makes you more productive and with it's big libraries, you can do stuff in just a single function! awesome! And to that, yes, we got the module storage working very well, and all we need to do is to figure out how will the modules be loaded and how do they work.

## New Idea

At first, I was thinking of using the same thing as the old OpenBlocks, creating extension points for modules to extend but with a different way of doing it. I was a bit skeptical on how will it turn out, since what openblocks is to "create your own stuff as you like". But with this method, we are heavily limited by the features we can extend and make! I mean, extension points are all determined by the app, it's going to be hard for modules to create their own features.

So, I scraped the old method and starting to do research about how to make something very modular and very extensible. After digging the internet for a few hours, I realised "wait, isn't the linux kernel is also modular? how do they work? and how can they work together?"

I was in search of learning on how the linux kernel modular system works, until I found out this youtube video explaining how the linux device driver modules works:

<iframe width="560" height="315" src="https://www.youtube.com/embed/juGNPLdjLH4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

I'm a fan of LiveOverflow but I didn't know he made a video about this, so I watched it till the end and I realised that the `/dev` directory is populated by modules, and those char files are used as a way to communicate to hardware without being bothered by writing the protocol to communicate with that hardware, all those are done by the kernel modules.

and it clicked! since other apps depend on char files on `/dev` to communicate with hardware (or possibly with the module itself), what if we have modules communicate with other modules using a single point? that's the idea, modules communicate to other modules through a communication method.

This idea might not seem interesting at all. But it's basically a way to communicate with other modules on runtime without knowing the othe other side it's connecting to, therefore, we can make stuff extensible without having them being hardcoded within the app!

You might be asking on how can the modules communicate with the app? is it through an API? no, we use the same communication method used to communicate to other modules, but use it to the app! cool!

<img src="/assets/img/communication.svg" style="background: white;" alt="communication"/>

Getting back to extensibility, Even though I have a different approach in doing this modular thing, I don't want to have the same thing where you can only extend a limited amount of components. I want people to extend every part of the app. So, what I have in mind is that the app will be completely feature-less! (except the module system part ofc). Every features, I mean EVERY FEATURES is going to be implemented by modules. I'm planning to have modules depend on other modules so we can have "replacability" (I don't even know if that's a word) for modules, where people can replace an other module implementation.

## Actual Implementation

Implementing modules is absolutely not hard at all, the hardest part is to figure out how to communicate from / to it. How the new openblocks modules work is very simple, it's simply a dexed code with reference to a library that's being shared with the module and the system, it will get loaded using the class `DexClassLoader` and be executed through that shared library.

<img src="/assets/img/module-loader-code.png" alt="Module loader code"/>

# End

Alright, I think that's all for this log, if you found something wrong with my blog, you can mail me about it on `me@nurihsanalghifari.my.id` and I'll happily fix it. Also, the new openblocks is going to be released soon! get ready for it! and thanks for being with us even after for a long time, I'm very sorry for the long wait, now it's the time to shine!
