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

The thing we need to know the most are the **local file headers**. They are the places where files are stored. They start with a signature of `0x04034b50` and follows a list of bytes containing information about the file such as the name, compressed, uncompressed sizes, the compression method, etc. After that, it is immediately followed by the file content.

<img alt="local file header" src="https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip-images/local-file-header.png"/>
<p style="font-size: 12px;" align="center">By Florian Buchholz - <a href="https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip.html">https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip.html</a></p>

Then, there are the **central directory headers**. They are placed right after the end of the file headers, they basically reference files that are in the zip file and provides a pointer that points the file header of the file. They start with a signature of `0x02014b50` and like headers do, follows a list of bytes that contains information abut the file that it's referencing.

<img alt="central directory header" src="https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip-images/central-file-header.png"/>
<p style="font-size: 12px;" align="center">By Florian Buchholz - <a href="https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip.html">https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip.html</a></p>

And at the end, there is the **end of central directory record**, usually shortened as "the EOCD". It's placed at the end of the file right after the final central directory header. It starts with the signature of `0x06054b50` and follows a list of bytes that contains information about the entries, where the central directory starts, comments, and many more.

<img alt="end of central directory record" src="https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip-images/central-file-header.png"/>
<p style="font-size: 12px;" align="center">By Florian Buchholz - <a href="https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip.html">https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip.html</a></p>


Pretty much all that zipalign does here is that it pads the extra field with null bytes so that the start of the file content is aligned to a 4-byte boundary (given that the file's compression method is set to `0x0000` / NONE).

### Demonstration

Suppose I have a regular zip file with the file `my_file` with the content `Hello world :)`:
<img alt="hex content of a simple zip file" src="/assets/zipalign-java/original-zip-file.png"/>

Let's walk through and classify which parts are which

#### End of central directory record

We know that the EOCD is always at the end, and also, we can see the start of the section just by its signature (which is `0x06054b50`). Signatures are basically IDs that tells what section starts!

<img src="/assets/zipalign-java/original-zip-file-eocd.png"/>

We can then retrieve the central directory start position field as a 4-byte int at offset `+0x10` relative from the start of EOCD.

<img src="/assets/zipalign-java/original-zip-file-eocd-cd.png"/>

Which we get `0x50` or `80` relative to the start of the file.

#### Central directory

Setting our pointer to `0x50`, it looks like it pointed to the start of the central directory; there are the bytes `0x50 0x4b 0x01 0x02` (which is a central directory signature)! awesome!

<img src="/assets/zipalign-java/original-zip-file-cd-start.png"/>

Here in the central directory, we're going to retrieve the compression method that is used to compress this file. If it's uncompressed, we'll then continue to align this file. The compression method field is a 2-byte number that lies at offset `+0xa` relative to the start of central directory.

<img src="/assets/zipalign-java/original-zip-file-cd-compression-method.png"/>

We get the value of `0x0000`, which means that it's uncompressed!

I mean, we can already clearly see that this file is clearly uncompressed, given the fact that we can see the content of the file just by viewing the ascii representation lol.

<img src="/assets/zipalign-java/original-zip-file-ascii-file-content.png"/>

Alright, since we know that this file is uncompressed, we're going to need to align it!

We must first understand what aligning a file entry in a zip file means.

> todo

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

