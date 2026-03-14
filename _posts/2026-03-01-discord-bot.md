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

After much consideration I finally gave up and sacrifised 5 dollars for a virtual machine hosted on [Vast.ai](https://vast.ai/) 

![vast_ai]({{ site.baseurl }}/assets/images/discord_bot/vast_ai.png "vast_ai")

 
It has *2* **RTX 3090** GPUs - this is one generation newer than my own PCs GPU, which it vastly outperforms:

|          | CUDA Cores | VRAM  | Processing power | Power consumption |
|----------|------------|-------|------------------|-------------------|
| RTX 2070 |    2304    |  8 GB |     7.5 TFLOPS   |      175 Watt     |
| RTX 3090 |    10496   | 24 GB |    35.6 TFLOPS   |      350 Watt     |

This should still be much cheaper than hosting on the large platforms like AVS, Azure or Google Cloud. This let me host a **Qwen3-32b** model for approximately 24 hours. I now have a better shot at this Danish text challenge.


## [](#personality)Personality
However unprofessional this might seem, I chose to shape the agent's personality after content from the [Newyawn](https://www.youtube.com/@Newyawn) youtube channel. It's a Danish guy doing voice over on movies such as Harry Potter. It has a lot of swearing, is very silly, but the humor is very iconic and had a huge impact on all group members' upbringing during our teenage years. Other than simply knowing the humor of Newyawn, I managed to transcribe 5 of my favorite videos such that it could be used to create the correct personality.

There are multiple ways to imbue a personality into an LLM. One way is to do supervised fine-tuning, which is not straight forward, but it is possible. A more common approach is to simply prompt it to be a certain way, by giving it clear instructions and at least a few examples. The challenge, however, is that if I were to simply format all the transcripts into the system prompt, each response would be quite slow. (...more on prompts + how to utilize the transcripts...)

Another way to design this "Flemse" character is to include this kind of humor in other aspects of the bot. What I haven't told you is that the bot does more than just act as a chat bot. It also contains some hard-coded instructions/responses such as the "help" command, or when it thinks. This is the result of asking for help with

```bash
!hjælp
```

```markdown
Hallå dér tøs, jeg er Flemming, men folk kalder mig også Flemse.
Jeg har følgendene kommandoer:
- `!hjælp` : Viser denne besked
- `!hej` : Jeg hilser på dig
- `!chat <dit input>` : Spørg mig om noget
- `!spurgt` : Så siger jeg bare noget crazy
```

And when the model is processing a query, the intermediate/loading message is defined by:
```python
silly_word = random.choice(silly_words)
loading_message = await ctx.send(f"⏳ Tænker på {silly_word}...")
```
meaning that I can define a list of silly words that match the personality I want to capture.




## [](#pipeline)LLM Pipeline
I quickly created a bot on the [Discord Developer Portal](https://discord.com/developers/applications) called Flemming (aka "Flemse"). 

- Simple discord bot example script
- Load model with Ollama
- Disable thinking (limit context of output length)
- Simple LangChain setup
  - add profile summaries of whoever is mentioned, into the system prompt



## [](#results)Results

- Admit that the sentences are somewhat gibberish
- I like how everything he says makes it sound like he has a stroke or is just very drunk
- Making Flemming quote videos at every interaction makes it veeery hard to get a proper output, especially as the model is not strong enough to weave the reference and user prompt together as a clever coherent output

