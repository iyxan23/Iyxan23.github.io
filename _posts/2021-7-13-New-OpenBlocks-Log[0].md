---
title: New OpenBlocks Log[0]
published: true
author: NurIhsan
date: 2021-07-13 07:36:47 +0000
categories: Projects New-OpenBlocks
tags: project kotlin android new-openblocks status
---

# Introduction

Pheww.. heres my first log of the "New OpenBlocks", today what we're going to do is to figure out how to compile stuff in android. Yesterday, I finished the a working simple code editor, so now we're going to need to know on how to compile source files into compiled files. This is my first log, note that I wouldn't like post logs everyday, I post them when I want to post them :)

Also, note that this log isn't supposed to be something like a reference, it's meant to be something like a timeline or story of what I did while developing "the new openblocks". Everything I said in these logs aren't prefectly accurate. If you find some mistakes you can drop a message on my email `me@nurihsanalghifari.my.id` and I'll happily fix them :)

# Trying to get ECJ to run

Alright, let's get our hands dirty on ECJ (Eclipse Java Compiler). It's a java compiler that's proven to work on android, It's being used on Sketchware and other IDEs made in android. From what I've heard, it's also more configurable than javac and also uses less memory, which is the best case since most android doesn't have a lot of memory compared to standard personal computers these days.

ECJ is written in java, With that, It should be very easy to get it running on android (errr, future Iyxan23 here, it's not). Also, I've discovered way earlier when developing on the old openblocks, there actually is ECJ depedency on maven that you can just use it right away, so I did that.

Running ECJ compiler directly is not that hard, you can use the class named `BatchCompiler` and use the function `execute`. In that execute function, you need to feed in some arguments to it, and these are it's arguments: The argument __of ecj__ (don't get confused by the function arguments), And two `PrintWriter`s: output (or more known as stdout) and err (stderr).

After implementing the function, it would look something like this in kotlin:

```kotlin
BatchCompiler.compile(command, PrintWriter(object : Writer() {
    override fun close() = l("Output writer closed")

    override fun flush() { }

    override fun write(cbuf: CharArray?, off: Int, len: Int) {
        cbuf?.let {
            l("[ECJ] ${String(it)}}")
        }
    }
}),
PrintWriter(object : Writer() {
    override fun close() = l("Error writer closed")

    override fun flush() {}

    override fun write(cbuf: CharArray?, off: Int, len: Int) {
        cbuf?.let {
            l("[ECJ ERR] ${String(it)}}")
        }
    }
}),
object : CompilationProgress() {
    override fun begin(remainingWork: Int) {
        l("Compilation started, remaining work: $remainingWork")
    }

    override fun done() {
        l("Compilation done!")
    }

    override fun isCanceled(): Boolean = false
    override fun setTaskName(name: String?) {
        l("Set task name: $name")
    }

    override fun worked(workIncrement: Int, remainingWork: Int) {
        l("Worked $workIncrement, Remaining work: $remainingWork")
    }
})
```
Oh yeah, that `l(String)` function is a helper function for logging stuff, so I wouldn't need to write long statements to log something.

This might not be the cleanest implementation (yes, I logged strings on the `write()` function instead of "writing" to a buffer and printing the buffer on `flush()`, why did I do it? Basically, I ran into some issues and thought that the printing mecanishm on flush was wrong, so I switched it onto the `write()` function, but then I realised it wasn't the case, so I actually fixed it but because I was lazy I left it there lmao).

I ran the app, put a java file to compile and... it crashed

I don't know why did it crashed, I thought it was because there was an exception thrown by ECJ, so I added the BatchCompiler.execute() function in a try block, and guess what, IT WORKS!! Just kidding it still crashed. I was thinking "what?", I'm vaguely catching Exception and it didn't catch the exception from ECJ? It must be some kind of internal error.

I ran the app on android studio and seeing from the logcat, ECJ was missing a runtime class named `javax.lang.model.SourceVersion` and my mind went aaaaaaaaaaAAAAAAAAAAAAAAAAAAAAAAAAA**AAAAAAAAAAAAAAAAAAAAAAAAAA**

![ECJ class not found screenshot](/assets/img/ecj-class-not-found.png)

After some searching, I realised that termux actually has ECJ in it's repository. So I downloaded ecj's deb file from termux's official packages repository. I extracted it and boom, there it is. Running ECJ is pretty simple, all you do is have a dexified ecj jar and a stub android jar (which I think contains the `javax.lang.model.SourceVersion` class) seeing from the bash script they used to run ecj:

```sh
dalvikvm -Xmx256m \
         -Xcompiler-option --compiler-filter=speed \
         -cp /data/data/com.termux/files/usr/share/dex/ecj.jar \
         org.eclipse.jdt.internal.compiler.batch.Main \
         -proc:none \
         -7 \
         -cp /data/data/com.termux/files/usr/share/java/android.jar \
         "$@"
```

Alright, we're on the right path!

So I replicated what termux has done, I made a function to extract the android.jar and ecj.jar included inside termux's ecj package (if it hadn't been extracted before). And also made a function that runs the command above, aaaaaand.... IT BROKE! jk, it works!!

![ECJ working](/assets/img/ecj-works.png)

Yeah pretty cool, got ecj up and runnin. If you want the project I made on this, it's quite simple, you can head to this [page](https://github.com/Iyxan23/ecj-android-example).

> Some of you might ask, "What, there's an error on the screenshot!". Yeah it does, but that error is because I was trying to compile a file that the app doesn't have a permission to access to (I'm on an android 11 emulator, it has the scoped storage restriction). But that doesn't matter, the ecj itself works and responds to my command correctly. (and yes I've tested it in compiling a file that the app has permission to, it works very well!)

Next log I'll continue doing dx and d8 (psssttt, they exist on termux's repository too, so it wouldn't be that long to implement)

I'll see you next time, have a great day!
