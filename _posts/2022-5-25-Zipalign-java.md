---
title: Implementing zipalign in java
author: NurIhsan
categories: Projects
tags: project sketchware software devlog
---

Finally, after five hundred years, Iyxan posted something to his blog!

Ehem.. So, cool, hi, I'm Iyxan and we're going to go through my journey in implementing zipalign in java! I just thought it's an interesting journey to be told.

It all started when Sketchware Pro was hit with an issue where the original zipalign binary doesn't seem to work well on certain devices. Me having looked at zipalign's source before, I thought it was somehow possible to port it to java, knowing that the main logic code (`ZipAlign.c`) was just a few hundred lines of code.

And that's where this journey starts!

I started off by trying to figure out what zipalign actually does, and I'm surprised that it's actually really simple!

Pretty much all that zipalign does is that it aligns uncompressed files on a zip to 4-byte boundaries. Confused? First we'll need to understand the structure of a zip file.

A regular zip file consists of 3 parts: Local file headers, central directory file headers, and the end of central directory record.

<img alt="zip internal layout from wikipedia" src="https://upload.wikimedia.org/wikipedia/commons/6/63/ZIP-64_Internal_Layout.svg"/>
<p style="font-size: 12px;" align="center">By Niklaus Aeschbache - http://www.enterag.ch/enterag/downloads/Zip64File_TechnicalDocumentation.pdf, Public Domain, https://commons.wikimedia.org/w/index.php?curid=19867337</p>

The thing we need to know the most are the local file headers. They are the places where files are stored. They start with a signature of `0x04034b50` and follows a [list of bytes](https://en.wikipedia.org/wiki/ZIP_(file_format)#Local_file_header) containing information about the file such as the name, compressed, uncompressed sizes, the compression method, etc. After that, it is immediately followed by the file content.

<img alt="local file header" src="https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip-images/local-file-header.png"/>
<p style="font-size: 12px;" align="center">By Florian Buchholz - https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip.html </p>

Pretty much all that zipalign does here is that it pads the extra field with null bytes so that the start of the file content is aligned to a 4-byte boundary (given that the file's compression method is set to `0x0000` / NONE).

Here's an example:

Suppose I have a regular zip file with the file `my_file` with the content `Hello world :)`:
<img src="todo"/>

This file is clearly uncompressed, given the fact that we can see the content of the file just by viewing the zip file. For computers to know that this file is uncompressed, we can see that the compression method byte (offset `+0x7`) is set to `0`.

So, we retrieve the position of where the file starts, which is `+0x1e + filename_length + extrafield_length`.
<img src="todo"/>

Filename length is at `+0x1a`, and extra field length is at `+01c`
<img src="todo"/>

Those values yields the file offset of `+0x1d + 7 + 28` at `0x41` or `65`. Which points exactly to the start of the file! ;)
<img src="todo"/>

Here we determine if this file needs to be aligned, which it does: `65 mod 4 = 1`!

It appears that it's off by three bytes to the next 4 byte boundary! so we increase the extra field length by `3` so it's `0x1f` or `31` and add 3 null bytes to the extra field.
<img src="todo"/>

And we're done!.. on the easy part..

