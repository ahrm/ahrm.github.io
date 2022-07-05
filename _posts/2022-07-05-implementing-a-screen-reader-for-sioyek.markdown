---
layout: post
title:  "Implementing text to speech for sioyek PDF viewer"
date:   2022-07-05 15:45:21 +0430
categories: jekyll update
---

Note: the scripts in this post were tested on windows and do have some windows-specific code, but they can easily be ported to other operating systems.

Here is the final result (enable audio):

<video muted autoplay controls width="100%">
    <source src="/images/2022-07-05-implementing-a-screen-reader-for-sioyek/tts.mp4" type="video/mp4">
</video>
# Introduction

One of the main new features in [sioyek 1.4](https://github.com/ahrm/sioyek) is the ability to execute external scripts and the ability to control `sioyek` from command line. In this post, we show how to combine this features to implement a simple (yet completely functional) screen reader for `sioyek`.


Sioyek has the ability to execute scripts, for example consider the following script which creates an OCRed version of a PDF file using [ocrmypdf](https://ocrmypdf.readthedocs.io/en/latest/) and then opens the result:

```python
import sys
import os

if __name__ == '__main__':
    file_path = sys.argv[1]
    new_path = file_path.split('.')[0] + '_new.pdf'
    os.system('ocrmypdf "' + file_path + '" "' + new_path + '"')
    os.system('sioyek "' + new_path + '"')
```

you can run it from `sioyek` by running the execute command and entering the following:
```
python /path/to/script.py "%1"
```
Here the `%1` expands to the path of the current file in sioyek. Note that the quotation marks are necessary if the path contains spaces. There are other expanded variables other than `%1`, here is the complete list:

* `%1` expands to the path of the current file
* `%2` expands to just the file name of the current file 
* `%3` expands to the selected text
* `%4` expands to the current page number
* `%5` expands to an input text which is received from the user using a text prompt
* `%6` expands to the text of the current line in `sioyek`'s [visual line mode](https://user-images.githubusercontent.com/6392321/168427739-007be805-a457-4d1f-ba14-35c5070aae5f.mp4)

Here is how it looks like in action:

<video muted autoplay controls width="100%">
    <source src="/images/2022-07-05-implementing-a-screen-reader-for-sioyek/ocr_simple.mp4" type="video/mp4">
</video>

Of course, typing this command every time is not a good solution, you can predefine commands in your `prefs_user.config` file:

```
execute_command_o python /path/to/script.py "%1"
```
Now instead of typing the command, you can run the `execute_predefined_command` command in sioyek (which itself can be bound to a key) and then press `o` (`o` is the name of the predefined command, you can have 26 predefined commands with names `a-z`). Or you could directly bind a key to execute `execute_command_o` in your `keys_user.config` file:
```
execute_command_o <S-r>
```
Note that the `o` is just the name of the command and doesn't have anything to do with its keybinding, for example here we have bound it to shift+r.

Here is another sample script which translates the highlighted text into french:

```python
import sys
from googletrans import Translator
from tkinter import messagebox
import tkinter

if __name__ == '__main__':
    text = sys.argv[1]
    translator = Translator()
    translation = translator.translate(text, dest='fr')
    root = tkinter.Tk()
    root.withdraw()
    messagebox.showinfo("tanslation", translation.text)
```
`prefs_user.config`:
```
execute_command_t python D:\sioyek-scripts\translate.py "%6"
```

`keys_user.config`:
```
execute_command_t <S-t>
```
Here is how it looks in action:

<video muted autoplay controls width="100%">
    <source src="/images/2022-07-05-implementing-a-screen-reader-for-sioyek/translate.mp4" type="video/mp4">
</video>

# Screen Reader

Here is a very simple text to speech scripts (works only on windows, can easily be ported to other operating systems by replacing windows text to speech with alternatives):

```python
import os
import sys

def escape(s):
    temp = "".join(c for c in s if ord(c) < 127)
    return temp.replace("'", "''")

if __name__ == '__main__':
    os.system('''PowerShell -Command "Add-Type -AssemblyName System.Speech; (New-Object System.Speech.Synthesis.SpeechSynthesizer).Speak('{}');'''.format(escape(sys.argv[1])))
```
`prefs_user.config`:
```
execute_command_t python D:\sioyek-scripts\tts.py "%6"
```

This is of course, very basic and requires the user to manually read every line but it is a good base and can easily be extended to include more advanced features. I implemented a more sophisticated version [here](https://github.com/ahrm/sioyek/tree/main/scripts/tts) which is too long to include in this post, but here is how it works at a high level:

* Instead of generating speech line-by-line (which would not flow very well), we concatenate all the lines and create an audio file for the whole page
* First, we generate a low-quality but fast audio file using windows tts and while that is playing we use mozilla's tts to generate a more high-quality sample. When the high-quality sample is ready, we swap it in.
* We align the audio and text using [aeneas](https://github.com/readbeyond/aeneas), when the user requests a read command from a specific line, we use this alignment to find the location of line within the audio file
* We automatically highlight the current line as it is being read

here is the relevant `prefs_user.config` file:
```
# start reading from highlighted line
execute_command_a python \path\to\server_read.py "%1" %4 "%6"
# stop reading
execute_command_b python \path\to\server_stop.py
# keep highlighting the current line being read
execute_command_c python \path\to\server_follow.py
# stop highlighting the current line being read
execute_command_d python \path\to\server_unfollow.py
# start the tts server (should be running before executing previous commands)
execute_command_e python \path\to\manager_server.py
```
and `keys_user.config` file:
```
execute_command_a r
execute_command_b <S-r>
execute_command_e <C-<f5>>
# when we manually move the line, stop following it
move_visual_mark_down;execute_command_d j
move_visual_mark_up;execute_command_d k
```

# Notes
Unfortunately mozilla tts is prone to something that I call "Spontaneous Stroke Syndrome" which is shown in the video below, I am not sure exactly what causes it, if someone has any ideas on what I may be doing wrong I would appreciate any help.


<video muted autoplay controls width="100%">
    <source src="/images/2022-07-05-implementing-a-screen-reader-for-sioyek/stroke.mp4" type="video/mp4">
</video>