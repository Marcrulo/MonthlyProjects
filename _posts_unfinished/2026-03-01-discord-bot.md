---
title: 13. LLM-powered Discord bot
description: Utilize the power of AI to create the newest member on the server
published: true
image: 'discord_bot/discord_icon.png'
---

## [](#prologue)Prologue
My favorite kind of projects when I started programming was to create these *Discord bots*. If you don't know Discord, it's more or less Slack/Teams/Zoom etc. for gamers. It was, and still is, very easy to create a bot that you can interact with. But back in the days, LLMs were not invented, so it was not really meant to chat with. Rather, it was either meant for silly little interactions or admin stuff for larger servers. 

Over these many years, my friends and I have accumulated a lot of interaction in the text channels, meaning that an LLM could learn about us quickly. So now I wanted to test its capabilities. And yes, we will again be facing danish data, meaning that bad grammar and reasoning is inevitable.

## [](#setup)Setup
With the high prices associated with GPU cloud hosting, it was very tempting to host the bot purely locally. The Discord bot API creates a tunnel for us, so we don't need to consider port-forwarding as a bottleneck. Buuuuut to my experience there exists no small models capable of generating coherent Danish text, so it was going to be hard. The contenders are the **Qwen3** and **Mistral** model families, as well as the **OpenEuroLLM-Danish** model (a Danish fine-tuned version of Gemma3). I ended up using the Qwen3 models and played around with model sized 4b,8b and 14b locally. Locally, 8b was the one that could run on the GPU without doing any CPU off-loading. 

After much consideration I finally gave up and sacrifised 5 dollars for a virtual machine hosted on [Vast.ai](https://vast.ai/) with *2* **RTX 3090** GPUs (one generation larger than my own PC, which it vastly outperforms):

|          | CUDA Cores | VRAM  | Processing power | Power consumption |
|----------|------------|-------|------------------|-------------------|
| RTX 2070 |    2304    |  8 GB |     7.5 TFLOPS   |      175 Watt     |
| RTX 3090 |    10496   | 24 GB |    35.6 TFLOPS   |      350 Watt     |

This should still be much cheaper than hosting on the large platforms like AVS, Azure or Google Cloud. This let me host a **Qwen3-32b** model for approximately 24 hours. I now have a better shot at this Danish text challenge.


## [](#personality)Personality

PERSONALITY
- prompts and references
- transcripts
- more than just prompts, its a feeling



## [](#pipeline)LLM Pipeline
I quickly created a bot on the [Discord Developer Portal](https://discord.com/developers/applications) called Flemming (aka "Flemse"). 

- Simple discord bot example script
- 


AGENT ARCHITECTURE
- langchain
- chains and caching
- discord and interaction




## [](#results)Results



OUTCOME
- funny chats
- other people's reactions and interactions

