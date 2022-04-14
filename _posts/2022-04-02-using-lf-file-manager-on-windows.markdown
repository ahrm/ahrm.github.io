---
layout: post
title:  "Using LF file manager on windows"
date:   2022-04-02 15:45:21 +0430
categories: jekyll update
---
![](/images/2022-04-02-using-lf-file-manager-on-windows/lf_main.jpeg)

[`lf`](https://github.com/gokcehan/lf) is an extremely fast and customizable terminal file manager for windows, mac and linux. By default `lf` doesn’t have many features that you might expect from a file manager (for example archiving and unarchiving files), but provides a powerful interface for the user to add these features themselves. Of course, the [documentation](https://pkg.go.dev/github.com/gokcehan/lf) has a long list of recipes for most common features that a user might want to add. However, the documentation and the tooling surrounding `lf` is mostly focused on linux, and setting it up on windows requires some modifications to the recipes provided in the documentation. In this post I will explain my journey to make lf the perfect file manager in windows. You can download all of the files and scripts detailed in this post [here](https://github.com/ahrm/dotfiles/tree/main/lf-windows). Note that these script are not meant to be copied verbatim (for example they contain some hard-coded paths which you may need to modify).

## Prequisites

I will not describe the basics of using lf in this post, the [documentation ](https://github.com/gokcehan/lf/wiki/Tutorial)has done an excellent job of that. I assume you are already familiar with the basics of lf.

In order to run all of the commands and scripts in this post, you will need python3, [`fzf`](https://github.com/junegunn/fzf/releases/tag/0.29.0), [`7zip`](https://www.7-zip.org/)and [`msys2`](https://www.msys2.org/). You will also need the following packages for python:

* `Pillow`

* `mupdf`

* `tkinter`

* `tkinterdnd2`

You can install all these packages using pip.

## Navigating the drives

As far as I know, by default in order to navigate to another drive in lf you have to type something like this: `:cd ‘D:\’` which is a lot of keystrokes for something as common as this. So I placed a mark with the name name at the root of each drive. For example after navigating to drive D, I marked it by pressing md and now I can jump to it by pressing `‘d`.

## Basic Utilities

Here we we configure renaming, quick reloading and other utilities (put the contents into your lfrc file)

    set filesep " "

    # quick rename using r
    cmd rename %sh -c 'mv -i %f% $0'
    map r push :rename<space>

    # reload config file using f5
    map <f-5> push :source<space>C:/Users/Lion/AppData/Local/lf/lfrc<enter>

    # use a and A to create files and directories
    cmd createfile %sh -c 'touch $0'
    cmd createdir %sh -c 'mkdir $0'
    map a push :createfile<space>
    map A push :createdir<space>

    # open explorer in current directory
    map S push &start.<enter>

    # copy file path
    map Y %echo %fx% | clip 

    # open file in nvim
    map V &nvim-qt %f%

    # archive management
    cmd zip %sh -c '7z a $0 %fx%'
    cmd extract_here %sh -c '7z e %f%'
    cmd extract_to %sh -c '7z e %f% -o$0'

## Fuzzy file search using `fzf`

lf wiki has a [section](https://github.com/gokcehan/lf/wiki/Integrations#fzf) for `fzf` integration, however the commands specified there are for linux and need some modification in order to work on windows. Which I have done and they are available [here](https://github.com/ahrm/dotfiles/tree/main/lf-windows/lf_scripts). Just copy [findfzf.bat](https://github.com/ahrm/dotfiles/blob/main/lf-windows/lf_scripts/findfzf.bat) and [fzfpy.py](https://github.com/ahrm/dotfiles/blob/main/lf-windows/lf_scripts/fzfpy.py) on your system and add the following to your `lfrc`.

    # use c-f to fuzzy search
    cmd fzf_jump push $python<space>D:/lf_scripts/fzfpy.py<space>%id%<enter>
    map <c-f> :fzf_jump

Of course, you have to replace `D:/lf_scripts/fzfpy.py` with the location of the file on your system (note that you probably need to edit `findfzf.bat` and specify the correct path of find.exe in your `msys2` installation).

## Drag and Drop

Being a terminal application, `lf` does not support drag and drop. However, some applications are impossible to use or very inconvenient without drag and drop. I wrote a [script](https://github.com/ahrm/dotfiles/blob/main/lf-windows/lf_scripts/drag.py) to add drag and drop functionality, just copy the script to your system and add the following to your `lfrc` (again, you have to modify the paths to point to your file locations).

    # drag and drop
    cmd drag push &python<space>D:/lf_scripts/drag.py<space>multi<space>%fx%<enter>

    # close the drag window after one use
    cmd dragonce push 
    &python<space>D:/lf_scripts/drag.py<space>once<space>%fx%<enter>
    map D push :dragonce<enter>

Here is how it looks like:

![Drag and Drop in lf](https://cdn-images-1.medium.com/max/2000/1*eQ1jzuO9jkN1C5jJWekGvw.gif)

## File Preview

lf is a 3 panel file manager: left panel shows the parent directory, middle panel shows the current directory and the right panel shows the contents of selected directory. If the selected item is a file instead of a directory, lf can show a preview of file on the right panel. By default it does so only for text files. I have written a preview script that displays some useful information for other file types. These include:

* File size and last modify date for all files

* Image dimensions for image files

* Number of pages and text content of the first page for PDF files

It looks like this:

![PDF file preview](https://cdn-images-1.medium.com/max/2000/1*D6Z7V-GwHeVEJg8euEYdKg.gif)

In order to activate it, you need to download [lf_preview.py](https://github.com/ahrm/dotfiles/blob/main/lf-windows/lf_scripts/lf_preview.py) and [preview.bat](https://github.com/ahrm/dotfiles/blob/main/lf-windows/lf_scripts/preview.bat) and add the following to your `lfrc`.

    # custom file preview
    set previewer "D:\\lf_scripts\\preview.bat"
