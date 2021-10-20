---
title: New OpenBlocks Log[1]
author: NurIhsan
date: 2021-07-22 05:07:14 +0000
categories: Projects New-OpenBlocks
tags: project kotlin android new-openblocks status
---
Welcome back to the log! Last few days there was a big event so I don't really did a lot of work on "The New OpenBlocks", so yeah, that's it, let's just get to the point.

# Short intro to dalvik
Some days ago I just finished implementing ECJ to "The New OpenBlocks", now what I'm going to do is to implement the dexers. "Dexers" are tools that translates from compiled java bytecode into dalvik bytecode. Why?, you might ask. It's because android doesn't have plain java  in it, it has a special bytecode called dalvik, this dalvik bytecode is optimized to run on low-end devices like android devices, that's why they used it (don't forget there is also another optimization done by android called ART in android Lollipop and up, but it's a subject I don't have a lot of knowledge in).

# Adding D8
Alright, what we're going to do is do the same thing as ECJ, _steal_ the dexed jar file from termux and use it on our app. But to my surprise, it's not there! :l. Well I guess I'm going to find the sources and compile it on my own, after some searching, I found the sources, they are from https://r8.googlesource.com/r8/ if you wanted to check them out. Reading from the docs in building, I seem to need to download depot\_tools. depot\_tools is a set of tools used by google devs to, I think deploy and do stuff with their repositories. I downloaded them on https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up and started to compile d8. It's very straightforward, you just run the `gradle.py` script inside depot\_tools with the `build.gradle` file included inside the repository.

After a little bit of waiting, It's finally compiled, so I head on to use d8 to dex itself hehe. Weird isn't to hear "dexing a dexer using itself", but yeah it's just a simple command
```
java -jar d8.jar --release --min-api 26 --output d8_dex.jar d8.jar
```
With that, we would see the file `d8_dex.jar` in the current directory we're in.

> Why use min api 26 (android 8)? Welp, because d8 uses the java.nio package (as I've heard from the Sketchware Pro discord server) and android doesn't have that package until api 26. That's also why we needed dx, it's so we can have support for older android versions.

After that, I tested the dexified d8 jar and as expected, it works!

# Short intro to aapt2
Next, we're going to try to implement aapt2, for those who doesn't know what it does is that `aapt2` is the one that manages the resources in APK files, hence the name "Android Asset Packaging Tool (2)". It will compile the `res` of a source code into a zip file filled with `.flat` (compiled resource) files. That compiled zip file will later be added with the APK.

## NativeBinariesManager
Since I've wrote the manager for native binaries, oh yeah I haven't told you people about this. So what `NativeBinariesManager` does is to simply manage the binaries carried within the app, because android 10+ removes the ability to execute files in the app's private directory (W^X Violation, for more, click [here](https://www.reddit.com/r/androiddev/comments/b2inbu/psa_android_q_blocks_executing_binaries_in_your/)), we're going to need to add them as native libraries (make sure to name it `lib(.+).so`) and use the `android:extractNativeLibs` flag to make the android system to extract the binaries to `context.applicaitonInfo.nativeDir` which will be executable. But, there is a catch, the flag `android:extractNativeLibs` is only for API 23+ (really, android). That's why we have `NativeBinariesManager`, it's so we can have one place where we can switch between using the legacy way of executing binaries, and using the new extraction without breaking the codes throughout the app.

The drawback of using the new technique is that you can't upgrade those native binaries without upgrading the entire app! which is aaaaaaaaAAAAAAAAAAAAAAAAAAA**AAAAAAAAAAAAAAAAAAAAA**. Android SUCKS

Oh yeah, another drawback is that when you see the new openblocks app gets released, you will see 2 types of APKs. Legacy and NativeLibs, each types will support for 4 CPU ABIS (arm64-v8a, armeabi-v7a, x86, and x86\_64), which would result in.. 8.. separate.. versions.. yeah android SUCKS.

Buuuuuut, Recently, the well-known user in our community usernamed "tyron" just made a discovery that an app that shares the same app private directory can symlink it's extracted native libs in the private directory and let the other app have access to it! Which is awesome, we can distribute the "native libraries apk" separately with the main APK! Solving the first issue.

Moving on from NativeBinariesManager, let's try to implement aapt2. Now currently, we're going to implement the legacy version because the new version is pain and I don't have a lot of knowledge in it. Implementing aapt2 would be a bit of a challange since aapt2 is a binary, rather than a jar file, we would need to compile aapt2 from it's source and target it for android so it would run in android, but because I'm lazy and I don't have experience in native stuff, I decided to use the build tools RohitVerma got from the web, you can see his [repo](https://github.com/RohitVermaOP/build-tools-for-android) here.

So yeah, with the legacy method, all I'm going to need to do is pack these binaries (aapt2 and zipalign) into a zip and extract it into a folder in the app private directory (for me, it would be on `$dataDir/binaries/`) and make them executable, Very straightforward.

After doing that, both binaries works fine as expected. Now I'm going to need to integrate it with the custom build system, which is p  a  i  n.

# Making a custom build system
What is a build system you might ask? It's a system that controls other programs in harmony so it would produce an output from a source code (building). In this case, we're remaking the android build system.

The core of the android build system is very simple, the task used to build an app are: ([SOURCE](https://stackoverflow.com/a/41140138/9613353), [2nd SOURCE](https://github.com/HemanthJabalpuri/AndroidExplorer))
 - Compile the resources using aapt2
 - Link the resources using aapt2
 - Compile the java sources into class files
 - Dex them using dx or d8 (I've mentioned these above)
 - Add all of those dexes with the res.apk generated by aapt during linking using ApkBuilder
 - zipalign the generated apk (not very necesary but why not)
 - finally, sign the apk using apksgner

That's it, quite simple right

## Integrating aapt2
Integrating aapt2 to the build system is not hard, because `NativeBinariesManager` already does the job for us, we can use it's one function to execute the aapt2 binary. aapt2 plays two roles in this building mecanishm, first is to compile the resources, and second is to link those resources into a res.apk file. Here are the commands used:

This is the command used to compile the app's res folder into a zip that contains .flat files (or simply, compiled res files)
```console
$ aapt2 compile --dir path/to/res/ -o res.zip # Compile the path/to/res/ folder into a res.zip file that contains .flat files
```

Now, this big command is used to link the resources, manifest, etc into one apk file
> Note: That `--java` parameter doesn't point to the java files, instead, it's the parameter where aapt2 will generate R.java in, cool right

```console
$ aapt2 link -I path/to/android.jar --auto-add-overlay --manifest path/to/AndroidManifest.xml --java path/to/gen/ --o res.apk res.zip
```

## Integrating ecj (the java compiler)
Integrating ecj is also not that hard, since ECJ is a jar, you could just add it as a dependency. But in this app, we're not going to, as I've described in the previous log, ecj for some reason doesn't work on the app. So I switched to using the dexed jar version and running it using `dalvikvm`. Heres the command used: 
```console
$ dalvikvm -Xmx256m -Xcompiler-option --compiler-filter=speed -cp path/to/ecj.jar org.eclipse.jdt.internal.compiler.batch.Main -proc:none -7 -cp path/to/android.jar -verbose -d path/to/outputFolder
```

## Integrating d8
As is said in the previous ones, this is not hard either. Since d8 is a jar, we can just use the dexed jar method like the one we used in ecj. Heres the command used:
```console
$ dalvikvm -Xmx256m -cp path/to/d8.jar com.android.tools.r8.D8 --release --classpath path/to/android.jar --output path/to/output/folder path/to/classfile.class path/to/another/classfile.class
```

But wait, d8 is a bit different. It doesn't recursively find classes inside a directory, which is a bit annoying. So you would need to recursively list every files in the class output folder and feed it into d8, other than that, it's simple as it is.

## Integrating ApkBuilder
Again, simple as before. ApkBuilder is a jar, instead of dexing it and using it separately, we're going to use it as a dependency instead. Why; you might ask, It's because running ApkBuilder as a command line program is deprecated and also because in the documentation, they recommended for people that wanted to create their own build system, [use ApkBuilder as a dependency instead](https://android.googlesource.com/platform/sdk/+/refs/heads/master/apkbuilder/readme.txt). So we did exactly that.

Using ApkBuilder is also not hard, they did a great job abstracting these functions so it would be easy for us to use it.
```kotlin
val apkBuilder = ApkBuilder(
    outApk, // the output apk
    resApk, // the res.apk aapt2 has generated on the link stage
    classesDex, // and the classes dex file generated by d8/dx
    null, // No key and no cert, we will sign this using ApkSigner instead
    null,
    printStream // verbose output, you can set this to null if you don't want it
)
apkBuilder.setDebugMode(false) // we don't want debug
apkBuilder.sealApk() // then finally, seal the apk
```
After this code is run, you would see an apk appear as pointed in the outApk variable

## Integrating zipalign
zipalign is not very necessary, but I mean why not. All it does is to optimize the apk and yada yada yada, it's a long explanation, check [this](https://developer.android.com/studio/command-line/zipalign) if you wanted to know more. This zipalign is quite the same as aapt2, it's a binary. So yeah, the steps are quite the same with aapt2. Here is the command used:
```console
$ zipalign 4 in.apk out.apk
```
Boom, very simple, nothing fancy

## Integrating ApkSigner
Even though we've built the apk, android wont install it because it hasn't been signed. It's like seeing a paycheck without any signature in it, noone knows who it's from or for who. So, we needed to sign the apk so the android operating system would trust our app. Because  creating a key is painful, we'r egoing to use a debug key to sign it instead. Debug key is a special key pre-made by android for debugging apps. Note that using debug key to sign your apk that will be distributed on a big scale is a no-no, you will need to create your own key for signing the apk.

Alright, because apksigner is also a jar, we're going to use the same method as ecj and d8 are, dex the jar and running it using dalvikvm.

Here is the command used to run ApkSigner:
```
$ apksigner sign --in in.apk --out out.apk --key path/to/privatekey.pk8 --cert path/to/publickey.x509.pem
```

And we're finally done! Open the apk and let the user enjoy it's own creation, yay!

and since we're also done, I'm also done doing this blog, see you on the next one, have a great day! Next blog I'm going to try to implement libraries for the app and probably modules (just probably, don't expect me to do it 100%)
