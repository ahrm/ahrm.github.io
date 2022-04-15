---
layout: post
title:  "Using Language Models to (probably) Read Faster"
date:   2022-04-14 15:45:21 +0430
categories: jekyll update
---

<style>
.button-selected{
    color: red;
}
.button-unselected{
}
img{
border: solid;
}

</style>
<script type="text/javascript">
function context_on_click(elem, num){
    let id = elem.id;
    var container = document.getElementById('context_comparison');

    var context_sizes = ['5', '10', '20', '40', '80', '160'];

    //var context_5_button = document.getElementById('context_5');
    //var context_10_button = document.getElementById('context_10');
    //var context_20_button = document.getElementById('context_20');
    //var context_40_button = document.getElementById('context_40');
    //var context_80_button = document.getElementById('context_80');
    //var context_160_button = document.getElementById('context_160');
    //var buttons = [context_5_button, context_10_button, context_20_button, context_40_button, context_80_button, context_160_button];

    var buttons = [];
    var images = []

    for (var context_size of context_sizes){
        buttons.push(document.getElementById('context_' + context_size));
    }

    for (var context_size of context_sizes){
        images.push(document.getElementById('img-context-' + context_size));
    }

    for (var button of buttons){
        if (button.id == elem.id){
            button.className = "button-selected"
        }
        else{
            button.className = "button-unselected"
        }
    }
    for (var image of images){
        if (image.id == "img-context-" + num){
            image.style = "";
        }
        else{
            image.style.display = "none";
        }
    }

    //container.innerHTML = '<img src="/images/2022-04-14-using-languge-models-to-read-faster/' + elem.id + '.png" width="100%"/>';
}

function method_on_click(elem){
    let id = elem.id;
    var container = document.getElementById('method_comparison');

    var bionic_button = document.getElementById('bionic');
    var unrefined_button = document.getElementById('unrefined');
    var refined_button = document.getElementById('refined');
    var filled_button = document.getElementById('fill');

    var bionic_image = document.getElementById('img-heuristic');
    var unrefined_image = document.getElementById('img-no-refine');
    var refined_image = document.getElementById('img-refine-no-fill');
    var filled_image = document.getElementById('img-refine-and-fill');

    var buttons = [bionic_button, unrefined_button, refined_button, filled_button];
    var images = [bionic_image, unrefined_image, refined_image, filled_image];
    for (var button of buttons){
        if (button.id == elem.id){
            button.className = "button-selected"
        }
        else{
            button.className = "button-unselected"
        }
    }
    for (var image of images){
        image.style.display = "none";
    }

    if (id == 'bionic'){
        bionic_image.style = "";
    }
    if (id == 'unrefined'){
        unrefined_image.style = "";
    }
    if (id == 'refined'){
        refined_image.style = "";
    }
    if (id == 'fill'){
        filled_image.style = "";
    }
}
</script>
## Idea
A couple of weeks ago I saw
[this](https://news.ycombinator.com/item?id=30787290) hackernews article about
a method of text rendering to increase text readability. The algorithm is
pretty simple: highlight the first few characters of each word (how many
characters depends on the size of the word). Here is a screenshot of what it
looks like from its [website](https://bionic-reading.com/):

<p align="center">
  <img src="/images/2022-04-14-using-languge-models-to-read-faster/bionic2.png" width="50%"/>
</p>

That got me thinking: what if instead of using a heuristic method to determine
how many characters to highlight, we used a language model? Specifically we
highlight the character only when the language model fails to predict the
character given its preceding context. Presumably if a language model is smart
enough to predict the character, so are we!

## Implementation
First of all, we need a character-based language model. I used a
single-character version of [reformer](https://arxiv.org/abs/2001.04451) fine
tuned for `enwiki8` dataset which is available on
[huggingface](https://huggingface.co/google/reformer-enwik8) (as I will mention in the notes section, this is a huge overkill but whatever, this is just an experiment ;) ). Let's test it:
```py
import torch
from transformers import ReformerModelWithLMHead

model = ReformerModelWithLMHead.from_pretrained("google/reformer-enwik8")

# removed for brevity, you can find them on the hugginface repo homepage
def encode(list_of_strings, pad_token_id=0): ...
def decode(outputs_ids): ... 

def generate_next_char(text, n_chars=1):
    return decode(model.generate(encode([text])[0],
                  max_length=len(text)+n_chars))

>>> generate_next_char("This is a ")
"This is a s"
>>> generate_next_char("This is a p")
"This is a pr"
>>> generate_next_char("This is a pr")
"This is a pro"
>>> generate_next_char("This is a pre")
"This is a prec"
>>> generate_next_char("This is a pred")
"This is a prede"
>>> generate_next_char("This is a predi")
"This is a predic"
>>> generate_next_char("This is a predic")
"This is a predict"
>>> generate_next_char("This is a predict")
"This is a predicti"
>>> generate_next_char("This is a predicti")
"This is a predictio"
>>> generate_next_char("This is a predictio")
"This is a prediction"
```
If we wanted to highlight the word "prediction" using the language model, it would look something like this:
**p**r**edi**ction, only the characters which language model got wrong are
highlighted. I implemented this in [sioyek](https://github.com/ahrm/sioyek) PDF
reader and the results look like this (if it looks blurry open the image in a new tab and zoom in):

<p align="center">
  <a href="/images/2022-04-14-using-languge-models-to-read-faster/no_refine.png"><img src="/images/2022-04-14-using-languge-models-to-read-faster/no_refine.png" width="100%"/></a>
</p>

I find sudden highlights in the middle of a word a little off-putting, let's change it so that a word is highlighted from the begining until the last mispredicted character. Using this scheme, **p**r**edi**ction would become **predi**ction (I call this process *refinement*). It looks like this in [sioyek](https://github.com/ahrm/sioyek):

<p align="center">
  <a href="/images/2022-04-14-using-languge-models-to-read-faster/refine_no_fill.png"><img src="/images/2022-04-14-using-languge-models-to-read-faster/refine_no_fill.png" width="100%"/></a>
</p>

It looks much better, but still words like **continue**d annoy me. If I have already read most of the word, there is little benefit in hiding the rest. So I changed it such that if more than 50% of a word is highlighted, we highlight the entire word (I call this process *filling*). It looks like this:

<p align="center">
  <a href="/images/2022-04-14-using-languge-models-to-read-faster/refine_and_fill.png"><img src="/images/2022-04-14-using-languge-models-to-read-faster/refine_and_fill.png" width="100%"/></a>
</p>

Here is a comparison of different highlight modes and the original (bionic) heuristic:
<div role="group" aria-label="Basic example" align="center">
  <button class="button-unselected" onclick="method_on_click(this);" id="bionic" type="button">Bionic</button>
  <button class="button-unselected" onclick="method_on_click(this);" id="unrefined" type="button">Unrefined</button>
  <button class="button-unselected" onclick="method_on_click(this);" id="refined" type="button">Refined</button>
  <button class="button-selected" onclick="method_on_click(this);" id="fill" type="button">Refined and Filled</button>
</div>

<p align="center" id="method_comparison">
  <img id="img-refine-and-fill" src="/images/2022-04-14-using-languge-models-to-read-faster/refine_and_fill.png" width="100%"/>
  <img id="img-refine-no-fill" src="/images/2022-04-14-using-languge-models-to-read-faster/refine_no_fill.png" width="100%" style="display: none;"/>
  <img id="img-heuristic" src="/images/2022-04-14-using-languge-models-to-read-faster/heuristic.png" width="100%" style="display: none;"/>
  <img id="img-no-refine" src="/images/2022-04-14-using-languge-models-to-read-faster/no_refine.png" width="100%" style="display: none;"/>
</p>

For performance reasons, instead of feeding the entire page from the begining to the point where I want to predict, I only feed the last `n` characters before the prediction points. Here is a comparison of the results for different values of `n`:

<div role="group" aria-label="Basic example" align="center">
  <button class="button-selected" onclick="context_on_click(this, '5');" id="context_5" type="button">5</button>
  <button class="button-unselected" onclick="context_on_click(this, '10');" id="context_10" type="button">10</button>
  <button class="button-unselected" onclick="context_on_click(this, '20');" id="context_20" type="button">20</button>
  <button class="button-unselected" onclick="context_on_click(this, '40');" id="context_40" type="button">40</button>
  <button class="button-unselected" onclick="context_on_click(this, '80');" id="context_80" type="button">80</button>
  <button class="button-unselected" onclick="context_on_click(this, '160');" id="context_160" type="button">160</button>
</div>

<p align="center" id="context_comparison">
  <img id="img-context-5" src="/images/2022-04-14-using-languge-models-to-read-faster/context_5.png" width="100%" />
  <img id="img-context-10" src="/images/2022-04-14-using-languge-models-to-read-faster/context_10.png" width="100%" style="display: none;"/>
  <img id="img-context-20" src="/images/2022-04-14-using-languge-models-to-read-faster/context_20.png" width="100%" style="display: none;"/>
  <img id="img-context-40" src="/images/2022-04-14-using-languge-models-to-read-faster/context_40.png" width="100%" style="display: none;"/>
  <img id="img-context-80" src="/images/2022-04-14-using-languge-models-to-read-faster/context_80.png" width="100%" style="display: none;"/>
  <img id="img-context-160" src="/images/2022-04-14-using-languge-models-to-read-faster/context_160.png" width="100%" style="display: none;"/>
</p>

It seems that we reach dimininshing returns at about 30 characters.

## Enabling in Sioyek
If you want to try these out on a PDF file, you can download the [latest experimental version of sioyek](https://github.com/ahrm/sioyek/releases/tag/2168768575). Here are the relevant configurations:
* `text_summary_url`: The url of the server which provides the summary. I did not include the server in sioyek itself because I did'nt want to bundle the entire pytorch with sioyek for an experimental feature. Instead I created a python script which runs a local server providing this feature. You can find the script [here](https://github.com/ahrm/sioyek/blob/main/scripts/summary_highlight_server.py). The default value is `http://localhost:5000/` which is the default port of the script, so if you don't change the script you don't have to set this value.
* `text_summary_should_refine`: 1 if you want refinement and 0 otherwise
* `text_summary_should_fill`: 1 if you want filling and 0 otherwise
* `text_summary_context_size`: number of characters in context for next character prediction

For example here is the relevant parts in my `prefs_user.config`:
```sh
text_summary_should_refine 1
text_summary_should_fill 1
text_summary_context_size 40
```

Of course we have default values for all of these configs so you don't have to change anything if you are comfortable with the default settings.


Now, in order to use this feature, run the `summary_highlight_server.py` script and then enable highlights in sioyek by executing `toggle_fastread` command (press `:` and type `toggle_fastread`, it may take a few seconds to compute highlights depending on your GPU).

## Notes and Improvements
* I don't have any data on whether this actually improves reading speed or not.
  But in my own subjective experience, I think it does.
* Currently this is too GPU-intensive to be deployed. Of course using a
  full-fledged language model for this task is overkill. Also, as mentioned in
  huggingface repo page, this model is not optimized for language generation.
  Probably the best option would be a relatively small RNN language model,
  however, I could not find any decent pre-trained character-based RNN language
  models and I don't have the resources to train it myself. Even simpler
  non-neural network models are probably good enough.
* One limitation of this approach is that we don't consider the future context
  to determine whether to remove a word. For example consider the snippet
  "task-specific training examples" (in our examples all three words were
  highlighted). But maybe if we knew that we were going to include both
  "task-specific" and "examples" then a language model could predict that the
  middle word in "task-specific [MASK] examples" is "training" with high
  probability and we could unhighlight the word "training". However, this is
  probably too computationally intensive to be worth it.
* Is it possible to use language models that use non-character tokens for this
  task? That would help a lot since most pre-trained language models are not
  character-based.
