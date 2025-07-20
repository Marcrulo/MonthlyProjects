---
title: 3. RAG model for Danish court trial data
description: A model for retrieving relevant historic trials, given short text description
published: true
---

## [](#prologue)Prologue
Prologue
Ethics regarding the use of AI for decision making in court, has been one of my favorite topics to discuss, as the philosophy of intelligence, conciousness, and emotions has had a big impact on my world view. 

As an engineer, I would also like to see things in practice, rather than just theorize about things. This is what drove me to start a project about AI with public Danish court trial data. Ideally, the available data would be structuered in a way similar to datasets in supervised learning, where we have a text description of the case, and a label corresponding to the punishment. Unfortunately, this was not the case, as the data consists of text documents that are not defined in a specific template. Therefore, the goal became to make a RAG model for retrieving these court documents based on a search prompt. I.e. to do natural text search on these documents. I want to do this using local models, and without LLM-pipeline libraries like LangChain.

As the ambition level of the project fell, I wanted to add a layer of complexity and utilize cloud computing for every step of the project, including testing, data processing, storage, and even endpoint hosting, primarily using Azure Machine Learning.

## [](#data-gathering)Data Gathering

In Denmark, we recently acquired a public database of court case documents, which are accesible through the website [link] or the API [link]. Although not exhaustive, it contains many cases from recent years. We will use the API to collect a local copy of all the data.

...We first need to authenticate:
[code]

...then get data...

request header (input)
- parameters and limitations
- [code]

request response (output)
- explain data overall
- more in-depth with the important ones


## [](#data-processing)Data Processing

So, we split the response data into  the meta data and the document data. The meta data can easily be described in a table, and will therefore be saved as a parquet-file (instead of csv) in order to save disk space. To further optimize space/memory usage, the column data types are selected manually.

before / after

This is a somewhat neglible addition, as the table is ~5000x10. But it's still a good principle, and I also just learned these tricks from my "Python and High-peformance Computing" course, so I wanted to see it in practice.

As for the actual documents, they are stored in HTML format. The text itself can easily be extracted using the Beautiful Soup (bs4) library. 
[Code]

It is also beneficial to split the text into small paragraphs, such that they can later be used for "chunking" (more on that later). It is easy to make these splits, as each actual paragraph is given by the HTML paragraph- or 'p' tag

Now, the most computationally demanding processing step is to convert each of these paragraphs into an embedding vector, which can take multiple hours. We explain the embedding vectors next.

Sentence Embeddings
Building a RAG model requires that we can convert our prompt, as well as the reference material, into a vector representation, such that they can be compared. This representation is formally called an embedding. We need ML embedding models to perform this transformation.

As we are working with a Danish dataset, it is not possible to simply use the default models, as they are usually in English. Even so-called "multi-lingual" models can still struggle with Danish, as the Danish language is quite underrepresented on the internet. My best attempt was to use a model from Huggingface that was explicitly fine-tuned for the Danish language [link]. It's definitely not perfect, but it's alright.

A limitation I have encountered is that the tokenization of words seems a bit wrong. In terms of vector similarity, the word "Kølle" ("club", as in the weapon) and "Køleskab" ("refrigerator") are way too similar. Even the vector representation of different types of blunt weapons does not even come close in terms of similarity.
example with cosine-similarity + words : in a table format

Let's get into a bit more details regarding this embedding model. 
- section about how embedding models are trained
- *embedding- and token dimension/size
...
Recall that our task is to find the trial document that best matches our prompt. We are quite limited by this token size, as we can't simply embed an entire document into a single vector (that would likely also require a larger embedding size). Instead, we will try and embed each paragraph of the documents, and track which paragraph belongs to which document. This is called "chunking", for which there are multiple strategies for how to embed these subtexts. We are lucky to have these beautifully split paragraphs, but for long texts, another strategy could be to embed overlapping subtexts of a fixed window-size, and a fixed overlap-size.

## [](#rag-model)RAG Model

Let's make it completely clear what we want to achieve using this model:

* A list of the k most relevant documents wrt. a prompt, given the similarity between the prompt and a single paragraph in these documents. (I have allowed duplicate documents to be shown)***

It is, however, quite common to extend a chat-bot with RAG capabilities, but not necessary. To do this, the RAG model should return some text, which can then be injected into the "system prompt" (not the "user prompt"). An example could be:
- show example of chatbot with RAG

If we were to do this ourselves, we should be cautious of:
1. The quality of the LLM used, due to challenges with the Danish language
2. Which information the RAG returns, as the entire reference document is too large to fit within the system prompt
Anyway, we are keeping it simple.

We have not talked about the challenge of searching for similar documents, which is very interesting. The straight forward way to find the most similar vectors wrt. the prompt vector, would be to simply check all options by brute force. In itself, this is slow, but using the FAISS (Facebook AI Similarity Search) library, many optimizations have been applied to speedup even the brute-force approach. It's very memory efficient, and it utilizes multiprocessing in order to parallelize the computation. They also offer approximate solutions for large datasets, where brute force in not feasible.

In the end, the flow looks like this:
- Flowchart/pipeline of entire model (later, update it to include the cloud)


## [](#cloud)Cloud
At first, the reason for using the cloud was primarily just to learn more about it, despite thinking that it won't be necessary, but it was actually quite beneficial. First of all, it enabled me to host an endpoint for the model, which makes it possible to interact with it from anywhere in the world. But most importantly, it was actually quite nice to offload the data processing to a cloud computer. Not only would my local PC have been unavailable for proper use, the computation also happened much faster, since the cheaper VMs ran the processing way faster than my own PC, and also did not encounter any crashes. 
(3 hours vs. 5+ hours ?)

I had initially started working on Machine Learning Operations (MLOps) at a DTU course of the same name, where we tried doing some simple ML workflows on Google Cloud Platform, but did not utilize the ML-specific services. Later on, I got some experience in Azure, working in the ML team at GN. At GN, I went through Microsoft's official tutorials, and finally got to test it out during this project. 

...

* Azure walkthrough
  * Storage
  * Environments
  * Azure ML notebooks
  * Train job script
* RAG endpoint
* Cost overview

## [](#results)Results
* Examples
  * Top 5 chunks
  * Best headline
  * Qualitative description of most similar document(s)
* Points in 2D space
* Present endpoint

## [](#reflection)Reflection
* RAG quality
* RAG limitation
* Improvement ideas
* Cloud usage and accessibility
  * Azure CLI in the future?
  * Local dev up online (slow feedback because of containers being built etc.)
  * How to manage Azure ML documentation, but ChatGPT is very good at the Azure CLI (and not so much with the UI)
  * Chat does know about debugging, though
  * There seems to have been some confusion regarding v1 and v2 of the Azure SDK

