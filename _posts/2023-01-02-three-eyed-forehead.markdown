---
layout: post
title:  "Three-eyed forehead in Stable Diffusion"
date:   2023-01-02 15:45:21 +0430
categories: jekyll update
---

<script type="text/javascript">

function onClickGenerator(elemId, path){
    let elem = document.getElementById(elemId);
    elem.src = path;
}

</script>

Today I saw an [interesting post](https://jalammar.github.io/ai-image-generation-tools/) on hackernews where the author tried to remake an old game by recreating the pixelated art using some AI image generation models for example:

<hr/>
<p align="center" >
  <img src="/images/2023-01-02-three-eyes/comparison.png" />
</p>
<hr/>
<p></p>

It worked reasonably well for most of the images, but there is one image which could not be easily created using the models, not even with stable diffusion inpainting:
<hr/>
<p align="center" >
  <img width="400px" src="/images/2023-01-02-three-eyes/nemesis.png" />
</p>
<hr/>
<p></p>

Apparently it was impossible to recreate the three eyes on the forehead. I have a few theories why this is the case:

* Probably there are a lot of "normal" looking humanoid portraits in the dataset, so the model is probably heavily biased towards producing "normal" humanoids.
* These models usually have trouble with numbers, so even when there are eyes in the forehead, it is rarely exactly three eyes

I was wondering if using the advanced inpainting of my [Stable Diffusion desktop frontend](https://github.com/ahrm/UnstableFusion#how-to-use-advanced-inpainting) frontend, we could achieve the illusive three-eyed forehead.

Before we begin, let me give you a quick overview of how stable diffusion inpainting (and my advanced inpainting implementation) work. I assume you are already familiar with the basics of how diffusion models work, if you are not, there are [excellent resources](https://stable-diffusion-art.com/how-stable-diffusion-work/) on the web.

In order to inpaint, first the masked portion of the image is filled in using a "dumb" inpainting algorithm (for example color each pixel with the closest non-masked pixel's color). Then we use the encoder to encode this image to the latent representation of the diffusion model. Then we add some noise to the masked part of the image and run the normal diffusion process.

Using advanced inpainting, we modify the first part of this process, so instead of using an algorithm to inpaint the missing parts, we could manually specify an initial image in the masked area. This heavily guides the diffusion process to generate something resembling the initial image. Here is a demo of this method:

<hr/>
<video muted controls width="100%">
    <source src="/images/2023-01-02-three-eyes/inpainting.mp4" type="video/mp4">
</video>
<hr/>
<p></p>

Here is how I approached this problem:
First I downloaded a random eye image from the web and used advanced inpainting to create a version with just one eye:


<hr/>
<p align="center" >
  <img src="/images/2023-01-02-three-eyes/one-eye.png" />
</p>
<div><small><i>Prompt: Demonic red eye on the forehead</i></small></div>
<div><small><i>Negative Prompt: Eyelashes</i></small></div>
<div><small><i>Generated using advanced inpainting by pasting an eye image from the web on the forehead</i></small></div>
<hr/>
<p></p>

I didn't bother making this look good, because we will have to inpaint over it anyway to generate the three eyes. I just needed something reasonable. Now we paste this eye on the forehead to create an initial image for the advanced inpainting:

<hr/>
<p align="center" >
  <img src="/images/2023-01-02-three-eyes/initial.png" />
</p>
<hr/>
<p></p>

Now we mask the three eyes, but use the original image as the initial image. This will guide the diffusion process to put three eyes in the masked location. We can even repeatedly apply this process, using the same mask each time but using the newer images (undoing changes if the new images were not as good), we can gradually guide it to generate something that we want (we can even change the prompt and parameters each time). Here is a sequence of generated images:


<hr/>
<p align="center" >
  <button onClick="onClickGenerator('seq', '/images/2023-01-02-three-eyes/initial.png')">0</button>
  <button onClick="onClickGenerator('seq', '/images/2023-01-02-three-eyes/1.png')">1</button>
  <button onClick="onClickGenerator('seq', '/images/2023-01-02-three-eyes/2.png')">2</button>
  <button onClick="onClickGenerator('seq', '/images/2023-01-02-three-eyes/3.png')">3</button>
  <button onClick="onClickGenerator('seq', '/images/2023-01-02-three-eyes/4.png')">4</button>
  <img id="seq" src="/images/2023-01-02-three-eyes/initial.png" />
</p>
<hr/>
<p></p>

You may notice the border around the masked area, but we could fix that we normal inpainting:


<hr/>
<p align="center" >
  <button onClick="onClickGenerator('mask', '/images/2023-01-02-three-eyes/masked.png')">masked</button>
  <button onClick="onClickGenerator('mask', '/images/2023-01-02-three-eyes/after.png')">after</button>
  <img id="mask" src="/images/2023-01-02-three-eyes/masked.png" />
</p>
<hr/>

And here is the final result:
<hr/>
<p align="center" >
  <img id="mask" src="/images/2023-01-02-three-eyes/after.png" />
</p>
<hr/>

Of course it is not a masterpiece, but it was a very fun experiment. And it has the potential to be way more fun, because I was running it on an old 1070, each inpainting took about 20 seconds which was quite annoying. But I could envision a future where generation is basically real-time, imagine navigating through possible generations using mouse wheel and tweaking the parameters and seeing the effects in real-time. With the [supposed improvements](https://twitter.com/emostaque/status/1598131202044866560) in stable diffusion, this future might not be far away.

<p></p>