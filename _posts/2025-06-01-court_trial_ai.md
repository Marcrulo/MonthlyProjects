---
title: 3. RAG model for Danish court trial data
description: A model for retrieving relevant historic trials, given short text description
published: true
---

## [](#prologue)Prologue
* Ethics on AI and Law
  * Interestinng type of data
  * Controversial tasks using AI
  * Data too unstructured to determine certain decisions made during a trial 
* Simpler, more manageble model - retrieval
* Need for cloud compute
  * Work requirement
  * Sharing machine learning models
  * Reduce gap
* Problem/project overview
  * Natural language search for trials
* Cloud assistance
  * Storage    (Azure    - blob storage)
  * Processing (Azure ML - Notebooks/Jobs)
  * Endpoint   (Azure ML - Endpoints) 

## [](#data-gathering)Data Gathering
* "Domstolsdatabasen"
  * Web portal vs API
  * API access
  * Describe data 

## [](#data-processing)Data Processing
* Fetch data
  * HTML and text
  * Meta data
  * Data compression (dtypes, parquet, embedded chunks)

## [](#sentence-embeddings)Sentence Embeddings
* Huggingface model (Danish limitations)
* Token-size limitation
* Chunking strategy

## [](#rag-model)RAG Model
* Puttings things together
* Embed prompt
* Embed historic data (chunks)
* Find similarity (FAISS indeces)
* How would this be extended to a chat-like model
  * "pre-prompt"
  * Find danish text-gen model

## [](#cloud)Cloud
* Embed in bulk (reason for cloud)
  * time-consuming
  * pc unavailable
  * faster cpus (mine ran for 5+ hours without terminating - Azure ran for 3)
  * Unexpected crashing (overload? writing big files?)
* Azure official tutorial
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

