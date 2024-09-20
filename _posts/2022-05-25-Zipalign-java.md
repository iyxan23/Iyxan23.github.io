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

<img alt="end of central directory record" src="https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip-images/end-of-central-directory-record.png"/>
<p style="font-size: 12px;" align="center">By Florian Buchholz - <a href="https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip.html">https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip.html</a></p>


Pretty much all that zipalign does here is that it pads the extra field with null bytes so that the start of the file content is aligned to a 4-byte boundary (given that the file's compression method is set to `0x0000` / NONE).

### Demonstration

Suppose I have a regular zip file with the file `my_file` with the content `Hello world :)`:
<img alt="hex content of a simple zip file" src="/assets/img/zipalign-java/original-zip-file.png"/>

Let's walk through and classify which parts are which.

#### EOCD

We know that the **EOCD** is always at the end, and also, we can see the start of the section just by its signature (which is `0x06054b50`). Signatures are basically IDs that tells what section starts!

<img alt="Original zip file EOCD" src="/assets/img/zipalign-java/original-zip-file-eocd.png"/>

We can then retrieve the central directory start position field as a 4-byte int at offset `+0x10` relative from the start of EOCD.

<img alt="Original zip file EOCD's Central Directory field" src="/assets/img/zipalign-java/original-zip-file-eocd-cd.png"/>

Which we get `0x50` or `80` relative to the start of the file.

Setting our pointer to `0x50`, it looks like it pointed to the start of the central directory; there are the bytes `0x50 0x4b 0x01 0x02` (which is a central directory signature)! awesome!

<img alt="The Central Directory field pointed by EOCD" src="/assets/img/zipalign-java/original-zip-file-cd-start.png"/>

#### Central Directory

Here in the **central directory**, we're going to retrieve the compression method that is used to compress this file. If it's uncompressed, we'll then continue to align this file. The compression method field is a 2-byte number that lies at offset `+0xa` relative to the start of central directory.

<img alt="Getting compression method from the central directory" src="/assets/img/zipalign-java/original-zip-file-cd-compression-method.png"/>

We get the value of `0x0000`, which means that it's uncompressed!

I mean, we can already see that this file is clearly uncompressed, given the fact that we can see the content of the file just by viewing the ascii representation lol.

<img alt="The ASCII content of the file shown" src="/assets/img/zipalign-java/original-zip-file-ascii-file-content.png"/>

Alright, since we know that this file is uncompressed, we're going to need to align it!

To align a file entry, we're going to need to retrieve the location of where the file entry starts. This field is located at `+0x2a` or `+42` in decimal relative to the start of the central directory.

<img alt="Getting the location of where the file entry starts" src="/assets/img/zipalign-java/original-zip-file-fh-offset.png"/>

With this, we know that the file entry is located on `0x00000000` relative to the file. (I mean we can already see it lol, but it'll point to file headers of zip files with multiple files)

#### Local File Header

Alright, let's get alinging it. All that we need to do is to make the start of the file content to align to 4-byte boundaries.

See this, the start of the content `Hello world :)` is not aligned to 4-byte boundaries highlighted by the white lines.

<img alt="Showing that the content of the file is not aligned to 4-byte boundaries" src="/assets/img/zipalign-java/original-zip-file-unaligned.png"/>

The trick here is that each local file header entries have a special field called the "extra field". This field is usually used to store extra information about a file or for a proprietary extension.

<img alt="Showing one of zip's field which is the extra field" src="/assets/img/zipalign-java/zip-format-extra-field.png"/>
<img alt="Showing one of zip's field which is the extra field" src="/assets/img/zipalign-java/zip-format-extra-field-show.png"/>

We simply extend the extra field just at the right length so that the file content that follows it aligns to 4-byte boundaries!

<img alt="Extending the extra field so the following content gets aligned" src="/assets/img/zipalign-java/original-zip-file-aligned-show.png"/>

And we're done!.. on the easy part..

Remember those offset fields in EOCD and central directory? They got changed because we've completely shifted the entire file starting at the start of the first file content. We're going to need to adjust them to point to the correct offsets so they are valid.

First is the central directory, we loop through them and modify the field which points to the file header that it's referencing. The field is located `+0x2a` or `42` in decimal relative to the start of the central directory with the length of a 4-byte int.

<img alt="Showing the the shifted file header" src="/assets/img/zipalign-java/aligned-zip-central-directory-file-offset.png"/>

Well, here we don't need to modify it because the shift happens after the file offset. In situations where you have multiple files, this is needed to be done.

And the EOCD, it has the field which stores the start of the central directory. It is a 4-byte int located `+0x10` or 17 in decimal relative to the start of the central directory header.

<img alt="Showing where the EOCD is" src="/assets/img/zipalign-java/aligned-zip-eocd-cd-offset.png"/>

We modify it to point to the start of central directory.

<img alt="Modifying the central directory the EOCD is pointing, because the file has shifted" src="/assets/img/zipalign-java/aligned-zip-eocd-cd-modify-offset.png"/>

And we're done! yay! that's pretty much all that needs to be done.

[You can head on to zipalign-java's source](https://github.com/Iyxan23/zipalign-java/blob/main/src/main/java/com/iyxan23/zipalignjava/ZipAlign.java#L80) to kind of compare this blog side-by-side and try to understand them better because I'm bad at words.
