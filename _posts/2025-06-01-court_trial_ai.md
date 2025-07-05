---
title: 3. RAG model for Danish court trial data
description: A model for retrieving relevant historic trials, given short text description
published: true
---

## [](#prologue)Prologue
* Ethics on AI and Law
* Simpler, more manageble model - retrieval
* Problem/project overview
* Cloud assistance

## [](#data-gathering)Data Gathering
* "Domstolsdatabasen"
  * Web portal vs API
  * Describe data 

## [](#data-processing)Data Processing
* Fetch data
  * HTML and text
  * Meta data
  * Data compression

## [](#sentence-embeddings)Sentence Embeddings
* Huggingface model
* Tokensize limitation
* Chunking strategy

## [](#rag-model)RAG Model
* Puttings things together
* Embed prompt
* Embed historic data (chunks)
* Find similarity (FAISS indeces)

## [](#cloud)Cloud
* Embed in bulk (reason for cloud)
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



