---
title: 1. Textual Inversion
description: Utilize textual inversion to learn an word-embedding of the Pokémon "Charizard", in order to generate an image with Stable Diffusion 1.5
published: true
---

## [](#prologue)Prologue
With the website quickly done, I still had time for another April project. I am currently taking a course on deep learning in computer vision, where we had about diffusion models for some weeks. A few weeks ago, I was not able to do the task on *textual inversion*. We were given most of the code, but the code stack was kind of a mess to get an overview of, and as a result, I could not generate anything remotely interesting. I was thinking to myself that it was not supposed to be that hard to implement, which is why I went to the internet. And as I should have thought, I was lead to [Hugging Face 🤗](https://huggingface.co)


## [](#textual-inversion)Textual Inversion
Let's start from scratch: "What is textual inversion?"

So, diffusion models like "Stable Diffusion" use (most notably) text to generate images. In order for these models to use that for anything, we need a model that learns comparisons between training images and their respective text labels. For this, a common solution is to use the CLIP model. This model encodes text and images in pairs, into a latent space, where the model tries to learn the encoding that increases similarity between image and text in each pair. Each **\<token\>** now has it's own latent representation that, through the prompt, guides how the diffusion model should produce an image. 

But what if I want to create an image of something very specific that can't be captured my a prompt? Do we then need to retrain the entire diffusion model? Luckily no, since the diffusion model and CLIP model are completely independent of each other. Instead, we use **textual inversion** to create a new embedding, that belongs to the same latent space that the CLIP encoder captures. This requires us to use a few images (5-7 should be plenty) that belongs to that embedding, in order to start.

To make it easy for ourselves, we can initialize a new embedding from an existing similar one. Then, we update the embedding directly, by optimizing wrt. the predicted noise of the diffusion model, similarly to how we'd normally train a diffusion model. This entire process should be done while freezing all parameters that are not the embedding.

This was a very brief overview. For more information, visit the original paper: [An Image is Worth One Word: Personalizing Text-to-Image Generation using Textual Inversion
](https://arxiv.org/abs/2208.01618)

![textualinversion]({{ site.baseurl }}/assets/images/textual_inversion/textual_inversion_paper.png "textualinversion")


> TLDR: I have some specific images of an object or style, and I want my diffusion model to learn to represent that. This is done by teaching the model a new embedding vector.

Alright now, let's get to work!





## [](#hugging-face)Hugging Face 🤗

A good place to start is with a [tutorial](https://huggingface.co/docs/diffusers/training/text_inversion) from Hugging Face, which forms the base of this (rather small) project. They provide us with the relevant textual-inversion 

- tutorial
- model limitations



## [](#dataset)Dataset

https://huggingface.co/docs/diffusers/training/create_dataset

- cat dataset
- charizard full
- charizard handpicked

## [](#training)Training

https://github.com/huggingface/diffusers/blob/main/examples/textual_inversion/textual_inversion.py

- local vs. cloud
- gradient checkpointing
- mixed precision
- xFormers
- deepspeed
- wandb


```python
os.environ["MODEL_NAME"] = "stable-diffusion-v1-5/stable-diffusion-v1-5"
os.environ["DATA_DIR"] = "./charizard"
os.environ["HUGGINGFACE_HUB_TOKEN"] = ...
```


```python
import torch
torch.cuda.empty_cache()

!accelerate launch textual_inversion.py \
  --pretrained_model_name_or_path    =   $MODEL_NAME \
  --train_data_dir                   =   $DATA_DIR \
  --output_dir                       =   "textual_inversion_charizard"
  --learnable_property               =   "object" \
  --report_to                        =   "wandb" \
  
  --placeholder_token                =   "<charizard>" \
  --initializer_token                =   "dragon" \

  --resolution                       =   512 \
  --train_batch_size                 =   1 \
  --gradient_accumulation_steps      =   8 \
  --max_train_steps                  =   2000 \
  
  --learning_rate                    =   1e-2 \
  --lr_warmup_steps                  =   100 \
  --scale_lr                         =   True \
  --lr_scheduler                     =   "cosine" \

  --gradient_checkpointing           =   True \
  --mixed_precision                  =   "fp16" \
  --enable_xformers_memory_efficient_attention \

  
```


## [](#results)Results
- prompting
- "charizard" vs "dragon"




```python
from diffusers import StableDiffusionPipeline

# Load the base model
pipeline = StableDiffusionPipeline.from_pretrained(
    "runwayml/stable-diffusion-v1-5", 
    torch_dtype=torch.float16
).to("cuda")

# Load your custom textual inversion embedding
pipeline.load_textual_inversion("textual_inversion_charizard", placeholder_token="<charizard>")

# Prompt model
prompt = "Something ... <charizard> ... something"
image = pipeline(prompt, num_inference_steps=50).images[0]
image.save("out.png")
```


```python
prompt = "<charizard>, a dragon-like Pokémon with blazing orange scales, roaring as it flies through a stormy sky. Flames burst from its mouth, lighting a volcanic land with lava and obsidian cliffs. Hyper-detailed digital art, vibrant colors, cinematic lighting, realism meets anime. ArtStation trending, 8K, dramatic scene with smoke swirling around its fiery tail"
```

```python
prompt = "A hyper-realistic cinematic illustration of <charizard> soaring through a dramatic sky, glowing embers around it, wings spread wide, powerful fire breath, epic lighting, golden hour, volumetric light, ultra-detailed, 4k, concept art, artstation, trending on ArtStation"
```

```python
prompt = "A breathtaking, ultra-detailed cartoon illustration of <charizard>, majestic and powerful, flying through a dramatic sunset sky, vibrant and saturated colors, intricate fire and smoke effects, cinematic lighting, highly polished fantasy art, masterpiece, 8k, extremely sharp and clean linework, professional character design"
```


## [](#reflection)Reflection
- quality
- what i learned
- things to try next time