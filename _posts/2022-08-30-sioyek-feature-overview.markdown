---
layout: post
title:  "Reading textbooks with lots of references using sioyek"
date:   2022-08-30 15:45:21 +0430
categories: jekyll update
---

This post is an overview of main features of [sioyek](https://sioyek.info/), a PDF viewer optimized for reading research papers and textbooks.

Suppose you are reading a textbook with a lot of references. Something like this:
<hr/>
<p align="center">
  <img src="/images/2022-08-30-sioyek-feature-overview/lots_of_references.png" />
</p>
<hr/>

<p></p>
Imagine how much time and context you lose by scrolling back and forth every time we see a reference. Sioyek automatically detects the reference
targets (even if the document doesn't have links, which is the case for the document in this example) and jumps to references. You can also mark your location before the jump so that you don't lose your context when you come back:


<hr/>
<video muted controls width="100%">
    <source src="/images/2022-08-30-sioyek-feature-overview/underline.mp4" type="video/mp4">
</video>
<hr/>
<p></p>

But wait, there is more! You don't even have to jump to the references because sioyek can show a preview of the referenced location:

<hr/>
<video muted controls width="100%">
    <source src="/images/2022-08-30-sioyek-feature-overview/preview.mp4" type="video/mp4">
</video>
<hr/>
<p></p>

But wait, there is more! The marker used in the first video to mark the line can also be moved to highlight the current line being read. This has many advantages:
* Makes the current line stand out, which makes it more readable, especially for people with dyslexia
* You never lose the context of which line you were reading (e.g. when someone calls you)
* Automatically handles multicolumn documents
* Since there is usually only one reference on the current line, we can automatically detect it and show the destination just by pressing a button, without even needing to click on the reference.

<hr/>
<video muted controls width="100%">
    <source src="/images/2022-08-30-sioyek-feature-overview/ruler.mp4" type="video/mp4">
</video>
<hr/>
<p></p>

But wait, there is more! You can search the papers in google scholar just by middle-clicking on their name. Or you can download them directly from google scholar and scihub by control+clicking on their name. And the beautiful thing is that last feature (downloading papers from google scholar and scihub) is not a built-in feature but it is implemented using an extension, and you can create similar extensions of your own!
[This](https://sioyek-documentation.readthedocs.io/en/latest/scripting.html) is the documentation on how to build your own extensions, and [these are](https://github.com/ahrm/sioyek-python-extensions) some of the extensions that I have built (including the one that downloads from google scholar and scihub).

<hr/>
<video muted controls width="100%">
    <source src="/images/2022-08-30-sioyek-feature-overview/paper_downloader.mp4" type="video/mp4">
</video>
<hr/>
<p></p>